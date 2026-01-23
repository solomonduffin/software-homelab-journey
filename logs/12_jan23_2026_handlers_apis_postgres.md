Part 1: The request. We call getUserGameHistory, async, and put in into a response. We can do whatever we want with that info.

```
const fetchGames = async () => {
      setGamesLoading(true);
      setGamesError(null);
      try {
        const response = await getUserGameHistory(5);
        // If response.games is null, default to empty array []
        const fetchedGames = response.games || [];
        setGames(fetchedGames);
        // If we got fewer than 5 games, there are no more to load
        setHasMore(fetchedGames.length === 5);
      } catch (error: any) {
        console.error('Failed to fetch game history:', error);
        setGamesError('Failed to load game history');
      } finally {
        setGamesLoading(false);
      }
    };


    fetchGames();
  }, [isAuthenticated, user]);
```

Part 2: The API call. We call an apiRequest with a specific URL to call for what we want. In this case, 
`/api/games/finished?limit=${limit}`

```
/**
 * Fetch user's completed games from the backend
 */
export async function getUserGameHistory(limit: number = 50): Promise<GameHistoryResponse> {
  try {
    console.log('Fetching game history...');
    const response = await apiRequest(
      `/api/games/finished?limit=${limit}`,
      'GET',
      undefined,
      true // requires auth to identify the user
    );


    console.log('Game history response:', response);
    return response;
  } catch (error: any) {
    console.error('Error fetching game history:', error);
    throw error;
  }
}
```

Part 3: The router. main.go handles pathing between requests and the handler.

```
// Public game viewing endpoints with optional auth (uses auth if present, otherwise requires user_id param)
    router.Handle("/api/games/finished", authMiddleware.OptionalAuth(http.HandlerFunc(handlers.GetUserFinishedGamesHandler))).Methods("GET")
```

Part 4: The handler. The handler reads from r *http.Request which has information in it about the request and who’s requesting. It writes information back in w, a stream. 
It calls the store with the desired method, which contacts and gets data from the database.


```
// GetUserFinishedGamesHandler retrieves a user's completed games
func (h *Handlers) GetUserFinishedGamesHandler(w http.ResponseWriter, r *http.Request) {
    if r.Method != http.MethodGet {
        http.Error(w, "Method not allowed", http.StatusMethodNotAllowed)
        return
    }


    // Try to get user from auth token first
    var userID string
    user, err := token.GetUserInfo(r)
    if err == nil && user.ID != "" {
        // User is authenticated, use their ID
        userID = user.ID
    } else {
        // No auth, check for user_id query parameter (for public access)
        userID = r.URL.Query().Get("user_id")
        if userID == "" {
            http.Error(w, "user_id query parameter is required when not authenticated", http.StatusBadRequest)
            return
        }
    }


    // Get limit from query parameter (default to 5)
    limitStr := r.URL.Query().Get("limit")
    limit := 5
    if limitStr != "" {
        if parsedLimit, err := strconv.Atoi(limitStr); err == nil && parsedLimit > 0 && parsedLimit <= 100 {
            limit = parsedLimit
        }
    }


    // Get finished games from database
    dbGames, err := h.postgresStore.GetUserFinishedGames(userID, limit)
    if err != nil {
        log.Printf("Error getting finished games for user %s: %v", userID, err)
        http.Error(w, "Internal server error", http.StatusInternalServerError)
        return
    }


...
```



Part 5: The store. Store is short for Data Store or Storage Layer. It is a specific folder (package) dedicated 100% to talking to the database. 
Its only job is to translate "Go requests" into "SQL commands.". It queries the database using PostgresStore object, which was opened by main.go for the purpose of speaking to the database. 
It gets the required information, stores it in a struct, and returns it to the handler.
```
// GetUserFinishedGames retrieves a user's completed games
func (p *PostgresStore) GetUserFinishedGames(userID string, limit int) ([]FinishedGame, error) {
    rows, err := p.db.Query(`
        SELECT
            id, white_player_id, black_player_id, game_mode, time_control,
            result, result_reason, white_elo_start, black_elo_start,
            starting_fen, moves, move_times, finished_at,
            -- Include opponent information
            CASE
                WHEN white_player_id = $1 THEN (
                    SELECT username FROM users WHERE id = black_player_id
                )
                ELSE (
                    SELECT username FROM users WHERE id = white_player_id
                )
            END as opponent_name,
            CASE
                WHEN white_player_id = $1 THEN 'white'
                ELSE 'black'
            END as user_color
        FROM finished_games
        WHERE white_player_id = $1 OR black_player_id = $1
        ORDER BY finished_at DESC
        LIMIT $2
    `, userID, limit)


    if err != nil {
        return nil, fmt.Errorf("failed to query finished games: %w", err)
    }
    defer rows.Close()


    var games []FinishedGame
    for rows.Next() {
        var game FinishedGame
        var movesJSON, moveTimesJSON *string


        err := rows.Scan(
            &game.ID,
            &game.WhitePlayerID,
            &game.BlackPlayerID,
            &game.GameMode,
            &game.TimeControl,
            &game.Result,
            &game.ResultReason,
            &game.WhiteEloStart,
            &game.BlackEloStart,
            &game.StartingFen,
            &movesJSON,
            &moveTimesJSON,
            &game.FinishedAt,
            &game.OpponentName,
            &game.UserColor,
        )
        if err != nil {
            return nil, fmt.Errorf("failed to scan finished game row: %w", err)
        }


        // Parse moves JSON array
        if movesJSON != nil && *movesJSON != "" {
            if err := json.Unmarshal([]byte(*movesJSON), &game.Moves); err != nil {
                return nil, fmt.Errorf("failed to parse moves JSON: %w", err)
            }
        } else {
            game.Moves = []string{}
        }


        // Parse move_times JSON array
        if moveTimesJSON != nil && *moveTimesJSON != "" {
            if err := json.Unmarshal([]byte(*moveTimesJSON), &game.MoveTimes); err != nil {
                return nil, fmt.Errorf("failed to parse move_times JSON: %w", err)
            }
        } else {
            game.MoveTimes = []int{}
        }


        games = append(games, game)
    }


    if err = rows.Err(); err != nil {
        return nil, fmt.Errorf("error iterating finished games rows: %w", err)
    }


    return games, nil
}
```



Part 6: Back to the handler. It gets that struct back, and converts it into a preferred format. Also does some smart calculating (like below it changes UserElo and OpponentElo based on UserColor. 
Then, it writes that information back into the open pipe ‘w’. This conversion process between raw DB data and pretty useable JSON data is called the "DTO Pattern" (Data Transfer Object).


```
    type FinishedGameResponse struct {
        ID           string    `json:"id"`
        Result       string    `json:"result"`
        ResultReason string    `json:"result_reason"`
        OpponentName string    `json:"opponent_name"`
        UserColor    string    `json:"user_color"`
        GameMode     string    `json:"game_mode"`
        TimeControl  string    `json:"time_control"`
        FinishedAt   time.Time `json:"finished_at"`
        UserElo      int       `json:"user_elo"`
        OpponentElo  int       `json:"opponent_elo"`
    }


    // Convert database models to response format
    games := make([]FinishedGameResponse, 0, len(dbGames))
    for _, dbGame := range dbGames {
        game := FinishedGameResponse{
            ID:           dbGame.ID,
            Result:       dbGame.Result,
            OpponentName: dbGame.OpponentName,
            UserColor:    dbGame.UserColor,
            GameMode:     dbGame.GameMode,
            TimeControl:  dbGame.TimeControl,
            FinishedAt:   dbGame.FinishedAt,
        }


        // Set result reason
        if dbGame.ResultReason != nil {
            game.ResultReason = *dbGame.ResultReason
        }


        // Set ELOs based on user color
        if game.UserColor == "white" {
            game.UserElo = dbGame.WhiteEloStart
            game.OpponentElo = dbGame.BlackEloStart
        } else {
            game.UserElo = dbGame.BlackEloStart
            game.OpponentElo = dbGame.WhiteEloStart
        }


        games = append(games, game)
    }


    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(map[string]interface{}{
        "games": games,
        "count": len(games),
    })
}
```


Part 7: Final. Back at fetchGames, we now have all of that information and we can use it how we want to.
```
const fetchGames = async () => {
      setGamesLoading(true);
      setGamesError(null);
      try {
        const response = await getUserGameHistory(5);
        // If response.games is null, default to empty array []
        const fetchedGames = response.games || [];
        setGames(fetchedGames);
        // If we got fewer than 5 games, there are no more to load
        setHasMore(fetchedGames.length === 5);
      } catch (error: any) {
        console.error('Failed to fetch game history:', error);
        setGamesError('Failed to load game history');
      } finally {
        setGamesLoading(false);
      }
    };


    fetchGames();
  }, [isAuthenticated, user]);
```



This line sets ‘games’ as the variable holding a user’s games, and ‘setGames’ as the method call to set the ‘games’ variable.

`const [games, setGames] = useState<FinishedGame[]>([]);`

Whenever setGames is called, React destroys the current view and re-paints the entire screen with the new data. This is called a "Re-render."


And finally, here we can see games being separated into each individual game and displayed on the screen (within a larger scrollView).
```
{games.map((game) => (
              <MatchRow
                key={game.id}
                result={formatGameResult(game)}
                opponent={game.opponent_name}
                date={formatGameDate(game.finished_at)}
                onPress={() => router.push(`/game/review/${game.id}`)}
              />
            ))}
```



**Analysis of the system**
#### The Good (Why this is solid):

1. **Separation of Concerns (The "Repository Pattern"):**
* Your `store` (Part 5) has **zero** clue that the web exists. It doesn't know what JSON is. It returns strict Go structs.
* Your `handler` (Part 6) has **zero** clue what SQL is. It doesn't know about tables or columns.
* **Why this wins:** If you decided tomorrow to switch your database from Postgres to MongoDB, you would *only* rewrite the `store` file. The `handler` and `main.go` wouldn't even notice.


2. **Frontend Service Layer:**
* In Part 2, you wrapped the API call in `getUserGameHistory`. Many juniors write `fetch('/api/...')` directly inside their React component.
* **Why this wins:** If the API URL changes, you update it in one file (`gameHistoryService.ts`), not in 50 different UI components.


3. **DTO Mapping (Part 6):**
* You are manually mapping `dbGame` to `FinishedGameResponse`.
* **Why this wins:** It prevents "Leaky Abstractions." If your DB has a column called `internal_user_hash_secret`, simpler frameworks might accidentally send that to the user just because it's in the database row. Your code explicitly filters exactly what gets sent.



#### The "Danger Zones" (Where to be careful):

1. **Raw SQL Strings (Part 5):**
* *Current setup:* `p.db.Query("SELECT ...")`
* *Critique:* Writing raw SQL strings is flexible but prone to typos. If you misspell `oppnent_name` in that string, you won't know until the app crashes at runtime.
* *Industry Standard Evolution:* As the app grows, Go developers often switch to tools like **sqlc** (which generates Go code *from* SQL) or query builders to catch these typos at compile time. For now, raw SQL is perfectly fine, just be careful.


2. **Manual JSON Mapping:**
* *Current setup:* The loop in Part 6 where you manually copy `ID: dbGame.ID`, `Result: dbGame.Result`, etc.
* *Critique:* It's tedious. If you add a field to the DB, you have to remember to add it to the Struct, the SQL query, AND the Handler mapping.
* *Verdict:* Keep doing it this way. It's tedious, but it's safe. "Magic" automappers often cause bugs.


### Final Verdict

You have successfully implemented a **Three-Tier Architecture**:

1. **Presentation Tier:** React/Expo (The UI)
2. **Application Tier:** Go Handlers (The Logic)
3. **Data Tier:** Go Store/Postgres (The Storage)

This is the gold standard for scalable web applications. You are ready to build the helper function.
