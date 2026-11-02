# SSE Tournament Bot

A Discord tournament-management bot for running individual and team competitions. It stores tournament data in MongoDB, creates private match channels, collects squad-power evidence and match proof, supports manager/reviewer decisions, tracks standings, and provides scheduling, series, brackets, tickets, exports, and audit tools.

> **Important:** This README documents the commands and behaviour present in this bot source. Discord may display command options in a different order. Use Discord's slash-command autocomplete as the final guide for each parameter. Features that depend on bot permissions (creating channels, changing roles, reading history, etc.) will fail if the bot lacks those permissions.

---

## Contents

1. [What the bot can do](#what-the-bot-can-do)
2. [Requirements and installation](#requirements-and-installation)
3. [Discord permissions and first setup](#discord-permissions-and-first-setup)
4. [How tournament formats work](#how-tournament-formats-work)
5. [Scoring and tie-breakers](#scoring-and-tie-breakers)
6. [Squad-power brackets](#squad-power-brackets)
7. [Individual and team tournaments](#individual-and-team-tournaments)
8. [Complete setup walkthrough](#complete-setup-walkthrough)
9. [Command reference](#command-reference)
10. [Match lifecycle, proof, appeals, and disputes](#match-lifecycle-proof-appeals-and-disputes)
11. [Scheduling and availability](#scheduling-and-availability)
12. [Standings, prizes, and completion](#standings-prizes-and-completion)
13. [Support tickets and audit logs](#support-tickets-and-audit-logs)
14. [Activation system](#activation-system)
15. [Environment variables](#environment-variables)
16. [Troubleshooting](#troubleshooting)

---

## What the bot can do

- Run **individual** or **team** tournaments.
- Use round-robin, single-elimination, double-elimination, Swiss, best-of series, or custom match arrangements.
- Create optional squad-power brackets with minimum and maximum power ranges.
- Collect a player's squad-power values and screenshot evidence during registration.
- Let managers approve/reject evidence or correct a verified average with a reason.
- Create teams, invite players, accept invitations, transfer captaincy, substitute members, and disqualify teams.
- Preview pairings before creating matches; generate or regenerate matchmaking.
- Create a private Discord channel for each automatically generated match, visible to the match participants and relevant tournament staff.
- Let participants submit match results and screenshots; let authorized staff approve, reject, or dispute results.
- Run best-of series and decide the series winner by the required number of game wins.
- Schedule matches, collect player availability, record forfeit/no-show outcomes, and handle appeals.
- Show leaderboards, team leaderboards, standings, player/team statistics, and finalized results.
- Configure first-, second-, and third-place prize descriptions and optional Discord award roles.
- Open private support tickets, manage ticket status, and export transcripts.
- Export tournament results to CSV and review administrative audit logs.

## Requirements and installation

### Software

- Python 3.11 or newer is recommended.
- A Discord application and bot token.
- A MongoDB database. MongoDB Atlas is suitable.
- The Python packages in `requirements.txt`.
- `activation.py` beside the tournament bot file if you are using the SSE activation system.

### Install packages

From the folder containing `bot.py` and `requirements.txt`, run:

```bash
python -m pip install -r requirements.txt
```

If you are using a `mongodb+srv://` Atlas URI, make sure `dnspython` is installed. It is included in the recommended requirements.

### Suggested requirements.txt

```txt
discord.py>=2.7.1
pymongo>=4.10.1
dnspython>=2.7.0
python-dotenv>=1.0.1
```

### Create the Discord application

1. Open the [Discord Developer Portal](https://discord.com/developers/applications).
2. Create an application and open its **Bot** page.
3. Create/reset the bot token and keep it private.
4. Open **General Information** and copy the numeric **Application ID**. This is used as `DISCORD_CLIENT_ID`.
5. Generate an OAuth2 invite URL with the `bot` and `applications.commands` scopes.
6. Give the bot the permissions described below before inviting it.

Never publish your bot token or MongoDB connection string. Do not commit `.env` to a public repository.

---

## Discord permissions and first setup

The bot needs the permissions relevant to the features you intend to use:

- View Channels
- Send Messages
- Embed Links
- Attach Files
- Read Message History
- Manage Channels (create and manage private match/ticket channels)
- Manage Roles (only if you want automatic placement award roles)
- Use Application Commands

The bot's highest role must be above any role it needs to assign as a tournament award. The bot also needs access to the category and channels where it will create or post content.

### Run `/setup`

A server administrator runs:

```text
/setup
```

The command creates or reuses a category named **SSE Tournaments** and creates/reuses these channels inside it:

| Channel | Purpose |
|---|---|
| `#announcements` | Tournament announcements and final placements by default |
| `#registration` | A suggested public place for registration instructions and discussion |
| `#discussion` | General tournament discussion |
| `#leaderboard` | A suggested place to share standings |
| `#match-directory` | A suggested directory for match information |
| `#support` | A suggested support landing channel; individual tickets are separate private channels |

The command stores the category and channel IDs in the bot's MongoDB configuration. The bot does not automatically post every leaderboard update into all these channels; use the appropriate public commands and announcement settings.

### Configure server-wide settings

- `/config show` — inspect saved channels and roles.
- `/config log-channel channel:<channel>` — select the audit/log channel.
- `/config support-role role:<role>` — select the role allowed to handle general support tickets.

`/setup` requires Administrator permission. The `/config` commands require Administrator or Manage Server permission.

---

## How tournament formats work

Choose the format when using `/tournament create`. The available values are `round_robin`, `single_elimination`, `double_elimination`, `series`, `swiss`, and `custom`.

### 1. Round robin (`round_robin`)

Every entrant is intended to play the other entrants. This format is useful when you want a league-style event where each player/team has multiple opportunities to play. The standings are based on the selected scoring method and tie-breakers.

**Important implementation note:** the automatic next-round generator in this bot explicitly handles single elimination, double elimination, and Swiss. For round-robin tournaments, use the match creation/scheduling tools to create the required fixtures. Do not assume `/matchmaking generate` automatically builds a complete round-robin fixture list.

### 2. Single elimination (`single_elimination`)

- Entrants are paired into matches.
- A winner advances to the next round.
- A loser is eliminated from the progression.
- A player/team without an opponent can receive an automatic **bye** and advance.
- The next round is generated after every match in the current round has a finalized result.

This is the fastest format for a knockout event. A draw blocks automatic progression because a winner is required.

### 3. Double elimination (`double_elimination`)

This format is designed around the two-loss elimination rule: an entrant should remain in contention after one loss and be eliminated after a second loss. It is more forgiving than single elimination.

**Important:** the bot's round generator includes double-elimination handling, but the exact bracket path and match placement are generated by its internal progression logic. Preview the pairings and inspect the resulting rounds before the event. For unusual bracket layouts or a manually controlled lower/upper bracket, use `custom` and create matches manually.

### 4. Best-of series (`series`)

A **series** is a set of games between the same two sides. It is not the same as a tournament round: one series can contain multiple individual match games, but the series winner is counted as one tournament win.

The default is **Best of 3 (Bo3)**:

- First side to win 2 games wins the series.
- If the score becomes 2–0, the third game is not needed.
- If the score becomes 1–1, a third game decides the series.
- With Best of 5, first to 3 wins; with Best of 7, first to 4 wins, and so on.
- The bot creates the next game in the series after the previous game has a finalized result, until one side reaches the required majority.
- A completed series gives the winning side one tournament win and 3 points; the losing side receives one loss. Individual games inside the series are not each counted as separate tournament wins.
- A series result can be appealed with `/match series-appeal` and resolved with `/match series-resolve`.

Configure the length before the tournament starts using `/tournament configure` and `series_best_of`. Valid values are 1–15; practical values are odd numbers such as 1, 3, 5, or 7. Best-of 1 is a single-game series.

**Example — Bo3:**

| Game | Result | Series score |
|---|---|---|
| Game 1 | Side A wins | A 1–0 B |
| Game 2 | Side B wins | A 1–1 B |
| Game 3 | Side A wins | A 2–1 B — Side A wins series |

A series appeal temporarily removes the series win/loss from live standings while it is under review. The head manager/reviewer can award the series to Side A, award it to Side B, or reject the appeal and keep the original winner.

### 5. Swiss (`swiss`)

Swiss pairs entrants with similar records over a number of rounds. Players/teams are not immediately eliminated after a loss.

- Each completed round is used to calculate records and points.
- The next round is generated after all matches in the current round are finalized.
- The pairing logic attempts to avoid repeat opponents and pair entrants with comparable scores.
- Set `swiss_rounds` to a positive number to choose the number of rounds.
- Set `swiss_rounds` to `0` for automatic rounds: the bot uses `ceil(log2(entrant_count))`, with a minimum of one round.
- A draw prevents automatic progression for the round; resolve the draw according to your tournament rules before expecting the next round.

Swiss is useful when you want a ranked field without eliminating everyone after their first loss.

### 6. Custom (`custom`)

Use this when managers want to control the fixture list themselves. Create matches with `/match create`, set maps and times, and manage each result normally. This is useful for invitationals, group stages, special formats, rematches, or manually designed upper/lower brackets.

---

## Scoring and tie-breakers

The tournament's `scoring` option determines the main ranking metric:

| Value | Meaning |
|---|---|
| `wins` | Rank by number of wins (default ranking approach) |
| `power` | Rank by verified squad power |
| `points` | Rank by points earned from results |
| `combined` | Calculate a weighted total from wins, power, and points |
| `win_percentage` | Rank by wins divided by wins + losses |

For ordinary finalized matches, a win adds **1 win and 3 points** to the winning player(s), while the losing player(s) receive **1 loss**. A draw adds 1 point to each side only where draws are allowed and recorded. In a series format, the series majority winner receives the win/points once the series is decided.

### Combined scoring

For `combined`, the bot uses:

```text
combined score = wins × win_weight
               + verified_power × power_weight
               + points × points_weight
```

Set the weights with `/tournament configure`. Choose weights that match your competition rules and explain them to players before registration opens.

### Tie-breakers

The default tie-breaker order is:

1. `wins`
2. `head_to_head`
3. `points`

You can change the order with `tie_breakers_csv` in `/tournament configure`. Other supported metrics used by ranking include `win_percentage` and `power`. For example:

```text
tie_breakers_csv: wins,head_to_head,points,power
```

Tie-breakers are applied in the order supplied. Make the rules public before the tournament starts.

---

## Squad-power brackets

Brackets are optional. Enable them when creating the tournament with `brackets:True`.

A bracket separates entrants into power ranges. For example:

| Bracket | Minimum | Maximum |
|---|---:|---:|
| Beginner | 0 | 3,000 |
| Intermediate | 3,001 | 5,000 |
| Advanced | 5,001 | No maximum |

The values are the average squad-power values calculated from the evidence submitted by a player. They are not automatically inferred from the game; players submit values and a manager verifies them.

### Bracket rules

- Brackets must be created before the tournament starts.
- Bracket ranges cannot overlap unless the manager explicitly sets `allow_overlap:True`.
- A bracket can have no maximum, meaning the range has no upper limit.
- Managers can edit a bracket, change its display order, or move a player to a chosen bracket before the tournament is locked.
- Each bracket can have its own standings and final placements.
- If brackets are disabled, participants compete in one overall pool.

Commands:

```text
/bracket add tournament_id:<ID> name:<name> min_power:<number> max_power:<number>
/bracket list tournament_id:<ID>
/bracket edit tournament_id:<ID> bracket_name:<name> new_name:<optional> min_power:<optional> max_power:<optional>
/bracket move-player tournament_id:<ID> player:<@player> bracket_name:<name> reason:<reason>
/bracket reorder tournament_id:<ID> bracket_name:<name> order:<number>
/bracket delete tournament_id:<ID> name:<name>
```

---

## Individual and team tournaments

### Individual mode

Each Discord user is an entrant. A player registers, submits squad-power values and evidence, and is paired against another player. The bot tracks the player's own wins, losses, points, and verified power.

### Team mode

A team is the entrant. A player first registers for the tournament, then creates or joins a team. The tournament sets minimum and maximum team size. The team captain can invite registered players, accept invitations, manage membership, and transfer captaincy.

Team power calculation can use:

- `average` — average of approved squad-power values from registered team members.
- `captain` — captain's approved power; if unavailable, the code falls back to the average of available approved member values.
- `all_members` — accepted as a configuration option; the implementation uses the approved member power values in team-power calculation, so managers should confirm their intended team-power policy and verify it in practice before relying on it for competitive seeding.

Other team controls include invite-only team membership, allowing incomplete teams, limiting one team per player or allowing multiple teams, kicking members, substituting a member, and disqualifying a team. Configure these before start.

---

## Complete setup walkthrough

This is the recommended order for a normal tournament.

### Step 1 — Set up the server

1. Invite the bot with `bot` and `applications.commands` scopes.
2. Give it the required channel and role permissions.
3. Run `/setup` as a server administrator.
4. Optionally set `/config log-channel` and `/config support-role`.

### Step 2 — Register a game

```text
/game create name:Mech Arena
/game list
```

The game is a server-level label used to group tournaments. Create it once, then select it when creating tournaments.

### Step 3 — Create the tournament

Example individual tournament:

```text
/tournament create
name:Mech Arena Solo Cup
game:Mech Arena
format:single_elimination
mode:individual
scoring:wins
brackets:True
manager_role:@Tournament Managers
description:Open solo competition
```

Example team tournament:

```text
/tournament create
name:Mech Arena Squad Cup
game:Mech Arena
format:series
mode:team
scoring:wins
brackets:True
manager_role:@Tournament Managers
team_size_min:3
team_size_max:3
team_power_rule:average
```

Discord will present these as slash-command options; you do not need to type the line breaks literally. Save the returned tournament ID. Most commands require this ID.

### Step 4 — Configure the tournament before registration/start

Use `/tournament configure` to set maps, start/end times, match deadlines, scoring weights, proof/review policy, Swiss rounds, series length, team rules, tie-breakers, and scheduling options.

Set the announcement channel:

```text
/tournament announcement-channel tournament_id:<ID> channel:#announcements
```

Set prize descriptions:

```text
/tournament prizes tournament_id:<ID> first_prize:... second_prize:... third_prize:...
```

Optionally assign placement roles:

```text
/tournament award-roles tournament_id:<ID> first:@1st second:@2nd third:@3rd
```

The bot needs Manage Roles, and its highest role must be above the award roles.

### Step 5 — Create brackets if enabled

```text
/bracket add tournament_id:<ID> name:Beginner min_power:0 max_power:3000
/bracket add tournament_id:<ID> name:Advanced min_power:3001
/bracket list tournament_id:<ID>
```

### Step 6 — Open registration

```text
/registration open tournament_id:<ID>
```

Players register using:

```text
/register tournament_id:<ID> squad_power_values:1200,1250,1300 proof:<screenshot> in_game_id:<optional-id>
```

The values are comma-separated numeric squad-power readings. Attach the requested screenshot evidence. The bot calculates the average and records the submission for manager review.

### Step 7 — Review squad-power evidence

Managers review each player:

```text
/squadpower review tournament_id:<ID> player:@Player decision:approve
```

Or reject with an optional note:

```text
/squadpower review tournament_id:<ID> player:@Player decision:reject note:Screenshot is unclear
```

If a verified average needs correction:

```text
/squadpower adjust tournament_id:<ID> player:@Player verified_average:1275 reason:Corrected after checking screenshot
```

A manager adjustment requires a reason and is audit-logged.

### Step 8 — Build teams (team mode only)

Players must be registered first. Then:

```text
/team create tournament_id:<ID> name:Team Alpha
/team invite tournament_id:<ID> team_name:Team Alpha player:@Player
/team accept tournament_id:<ID> team_name:Team Alpha
/team list tournament_id:<ID>
```

A player may also use `/team join` where the tournament's team settings permit joining without an invitation.

### Step 9 — Close registration

```text
/registration close tournament_id:<ID>
```

If registration needs to be reopened before matches exist, use `/registration reopen`. If matches already exist, the bot requires managers to regenerate unstarted matchmaking first.

### Step 10 — Preview and generate matches

Preview first:

```text
/matchmaking_preview tournament_id:<ID>
```

If the pairings look correct:

```text
/matchmaking tournament_id:<ID>
```

For formats or schedules that need manual fixtures, use `/match create` instead. Matchmaking creates private channels for generated matches when Discord permissions allow it.

### Step 11 — Start the tournament

```text
/tournament start tournament_id:<ID>
```

The tournament must have registration closed and at least one match generated/created. Starting locks important settings. Match results and rounds can now progress.

### Step 12 — Play, submit, and review matches

Participants play their match, then submit the result and screenshot proof:

```text
/match submit match_id:<MATCH_ID> winner_side:A proof:<screenshot>
```

An authorized manager/reviewer checks the evidence:

```text
/match review match_id:<MATCH_ID> decision:approve note:Verified
```

Use `reject` or `dispute` when appropriate. For automatically progressing formats, the next round is generated after all required matches in the current round have finalized.

### Step 13 — Publish results and finish

```text
/leaderboard tournament_id:<ID>
/tournament results tournament_id:<ID>
/tournament finish tournament_id:<ID>
/tournament winners tournament_id:<ID>
```

Finishing freezes the final standings. The bot attempts to announce final placements in the configured announcement channel and assign configured award roles.

---

# Command reference

`<required>` means the option must be supplied. `[optional]` means it can be left blank. Role/user/channel options should be selected from Discord's picker where possible. All IDs are the IDs returned by the bot, not the tournament's display name.

## General commands

| Command | What it does |
|---|---|
| `/help` | Shows a short command overview. |
| `/setup` | Creates/reuses the SSE Tournaments category and base channels. Administrator required. |
| `/announce tournament_id:<ID> message:<text> [channel:<channel>]` | Posts a tournament announcement embed. Uses the configured announcement channel if no channel is selected. |
| `/leaderboard tournament_id:<ID> [bracket_name:<name>]` | Shows player standings, optionally for one bracket. |
| `/team_leaderboard tournament_id:<ID> [bracket_name:<name>]` | Shows team standings. |
| `/standings tournament_id:<ID> [bracket_name:<name>]` | Shows wins, losses, points, and win rate. |

## Game commands — `/game`

| Command | What it does |
|---|---|
| `/game create name:<name>` | Creates a game label. Requires Manage Server. |
| `/game list` | Lists active games in this server. |
| `/game edit current_name:<name> new_name:<name>` | Renames an existing game. Requires Manage Server. |
| `/game delete name:<name>` | Deactivates a game; it does not erase tournament history. Requires Manage Server. |

Create a game before creating a tournament that uses it.

## Tournament commands — `/tournament`

| Command | What it does |
|---|---|
| `/tournament create` | Creates a draft. Required options: `name`, `game`, `format`, `mode`, `scoring`, `brackets`, `manager_role`. Optional: `description`, `team_size_min`, `team_size_max`, `team_power_rule`. |
| `/tournament list` | Lists recent non-archived tournaments and their IDs/status. |
| `/tournament edit tournament_id:<ID> [name:<name>] [description:<text>]` | Changes name/description before settings are locked. |
| `/tournament configure` | Updates maps, dates, deadlines, scoring, result policy, Swiss/series settings, team settings, tie-breakers, and schedule controls. See detailed options below. |
| `/tournament clone tournament_id:<ID> new_name:<name>` | Creates a new draft using the source tournament's settings. Does not copy registrations or matches. |
| `/tournament start tournament_id:<ID>` | Starts the tournament after registration is closed and matches exist; locks settings. |
| `/tournament pause tournament_id:<ID> [reason:<text>]` | Pauses the tournament and stops result submissions/reviews. |
| `/tournament resume tournament_id:<ID>` | Resumes a paused tournament. |
| `/tournament finish tournament_id:<ID> [force:True/False]` | Completes the tournament and freezes final standings. `force` allows finishing with unresolved matches; use carefully. |
| `/tournament cancel tournament_id:<ID> reason:<text> confirm:True` | Cancels the tournament. Confirmation is required. |
| `/tournament archive tournament_id:<ID>` | Archives a completed or cancelled tournament. |
| `/tournament delete tournament_id:<ID> confirm:True [delete_channels:True/False]` | Permanently deletes tournament records. Related channels are archived by renaming unless `delete_channels` is true. Disputed matches block deletion. |
| `/tournament announcement-channel tournament_id:<ID> channel:<channel>` | Sets the announcement/final-placement channel. |
| `/tournament results tournament_id:<ID> [limit:<1–25>]` | Lists finalized match results. |
| `/tournament winners tournament_id:<ID>` | Displays frozen final placements. |
| `/tournament prizes tournament_id:<ID> [first_prize:<text>] [second_prize:<text>] [third_prize:<text>]` | Saves prize descriptions for the final announcement. |
| `/tournament add-manager-role tournament_id:<ID> role:<role>` | Adds a role that can manage this tournament. |
| `/tournament remove-manager-role tournament_id:<ID> role:<role>` | Removes a tournament manager role. |
| `/tournament add-reviewer-role tournament_id:<ID> role:<role>` | Adds a role that can review results. |
| `/tournament remove-reviewer-role tournament_id:<ID> role:<role>` | Removes a result-reviewer role. |
| `/tournament add-head-manager-role tournament_id:<ID> role:<role>` | Adds a head-manager role for escalated disputes/series appeals. |
| `/tournament remove-head-manager-role tournament_id:<ID> role:<role>` | Removes a head-manager role. |
| `/tournament award-roles tournament_id:<ID> [first:<role>] [second:<role>] [third:<role>] [clear_existing:True/False]` | Sets Discord roles for final placements. Configure before start. |

### `/tournament configure` options

All options except `tournament_id` are optional. Leave an option blank to keep its current value.

| Option | Purpose |
|---|---|
| `maps_csv` | Comma-separated map names, e.g. `MVP,Factory,Control Tower`. |
| `start_at`, `end_at` | Tournament date/time in ISO-8601 format with timezone. Example: `2026-11-10T18:30:00+05:30`. |
| `match_deadline_hours` | Number of hours allowed for a match deadline (1–720). |
| `scoring` | `wins`, `power`, `points`, `combined`, or `win_percentage`. |
| `proof_required` | Whether match proof is required. |
| `approval_required` | Whether a manager/reviewer must approve submitted results. |
| `allow_draws` | Whether draws are allowed. |
| `swiss_rounds` | Swiss round count, 0 for automatic, or 1–20. |
| `series_best_of` | Best-of length, 1–15; normally use an odd number. |
| `allow_incomplete_teams` | Whether teams below the configured maximum/target size can proceed. |
| `team_invites_only` | Restricts team joining to invitation flow. |
| `tie_breakers_csv` | Comma-separated tie-breaker order, e.g. `wins,head_to_head,points,power`. |
| `win_weight`, `power_weight`, `points_weight` | Weights used by combined scoring (0–100 each). |
| `team_power_rule` | `average`, `all_members`, or `captain`. |
| `team_size_min`, `team_size_max` | Team size boundaries (1–20). |
| `allow_multiple_teams` | Whether a player may be in multiple teams in this tournament. |
| `leaderboard_update_hours` | Configured leaderboard update interval (0–168; 0 disables scheduled updates). |
| `match_duration_minutes` | Expected match duration (15–240 minutes), used by scheduling logic. |

Dates must include a timezone offset or `Z` for UTC. Do not enter a local date/time without a timezone.

## Registration commands — `/registration` and `/register`

| Command | What it does |
|---|---|
| `/registration open tournament_id:<ID>` | Opens registration from draft/registration-closed state. |
| `/registration close tournament_id:<ID>` | Closes registration. |
| `/registration reopen tournament_id:<ID>` | Reopens registration before start, subject to existing-match restrictions. |
| `/registration remove-player tournament_id:<ID> player:<user> reason:<text> confirm:True` | Removes a player from registration. Confirmation and reason required. |
| `/registration disqualify-player tournament_id:<ID> player:<user> reason:<text>` | Disqualifies a player while retaining evidence/match history. |
| `/register tournament_id:<ID> squad_power_values:<numbers> proof:<attachment> [in_game_id:<ID>]` | Registers the user and submits squad-power values plus screenshot evidence. |
| `/unregister tournament_id:<ID>` | Withdraws the current user from the tournament. |

Submit multiple power readings separated by commas, for example `1200,1250,1300`. The bot calculates an average; managers decide whether the evidence is acceptable.

## Squad-power commands — `/squadpower`

| Command | What it does |
|---|---|
| `/squadpower review tournament_id:<ID> player:<user> decision:approve/reject [note:<text>]` | Approves or rejects the player's submitted power evidence. |
| `/squadpower adjust tournament_id:<ID> player:<user> verified_average:<number> reason:<text>` | Sets the verified average manually with an audit reason. |

## Bracket commands — `/bracket`

| Command | What it does |
|---|---|
| `/bracket add tournament_id:<ID> name:<name> min_power:<number> [max_power:<number>] [allow_overlap:True/False]` | Adds a squad-power range. |
| `/bracket list tournament_id:<ID>` | Lists brackets and their IDs/ranges. |
| `/bracket edit tournament_id:<ID> bracket_name:<name> [new_name:<name>] [min_power:<number>] [max_power:<number>] [allow_overlap:True/False]` | Edits a bracket. |
| `/bracket move-player tournament_id:<ID> player:<user> bracket_name:<name> reason:<text>` | Moves a player to a selected bracket. |
| `/bracket reorder tournament_id:<ID> bracket_name:<name> order:<1–50>` | Changes bracket display order. |
| `/bracket delete tournament_id:<ID> name:<name>` | Deletes a bracket before the tournament is locked. |

## Team commands — `/team`

| Command | What it does |
|---|---|
| `/team create tournament_id:<ID> name:<name>` | Creates a team; the creator becomes captain. |
| `/team join tournament_id:<ID> team_name:<name>` | Requests/joins a team where settings allow it. |
| `/team list tournament_id:<ID>` | Lists teams. |
| `/team edit tournament_id:<ID> current_name:<name> [new_name:<name>] [logo_url:<URL>]` | Renames a team or changes its logo URL. |
| `/team invite tournament_id:<ID> team_name:<name> player:<user>` | Captain invites a registered player. |
| `/team accept tournament_id:<ID> team_name:<name>` | Accepts a team invitation. |
| `/team leave tournament_id:<ID> [team_name:<name>]` | Leaves the current team. |
| `/team delete tournament_id:<ID> team_name:<name> confirm:True` | Disbands a team before team lock. |
| `/team kick tournament_id:<ID> team_name:<name> player:<user> reason:<text>` | Removes a player from a team. |
| `/team transfer-captain tournament_id:<ID> team_name:<name> new_captain:<user>` | Transfers team captaincy. |
| `/team substitute tournament_id:<ID> team_name:<name> outgoing:<user> incoming:<user> reason:<text>` | Replaces a team member with another registered player. |
| `/team disqualify tournament_id:<ID> team_name:<name> reason:<text>` | Disqualifies a team. |

Team actions are subject to captain, manager, registration, team-size, and tournament-lock rules. A player must generally be registered before joining a team.

## Match commands — `/match` and matchmaking

| Command | What it does |
|---|---|
| `/matchmaking_preview tournament_id:<ID>` | Shows proposed pairings without creating match channels. |
| `/matchmaking tournament_id:<ID>` | Generates initial pairings and private match channels. |
| `/matchmaking_regenerate tournament_id:<ID> confirm:True` | Regenerates unstarted matches after confirmation. Do not use after matches have begun. |
| `/match create tournament_id:<ID> side_a:<entrant> side_b:<entrant> [scheduled_at:<ISO time>] [map_name:<name>] [bracket_name:<name>]` | Manually creates a match for custom formats/rematches. |
| `/match view match_id:<ID>` | Shows match details and status. |
| `/match repair-channel match_id:<ID>` | Recreates a missing private match channel. |
| `/match edit match_id:<ID> [map_name:<name>] [deadline_at:<ISO time>]` | Edits match map/deadline. |
| `/match reschedule match_id:<ID> scheduled_at:<ISO time> reason:<text> [force:True/False]` | Changes match time with a reason. |
| `/match schedule-all tournament_id:<ID> [confirm:True]` | Automatically schedules unscheduled matches inside the configured tournament date window. |
| `/match submit match_id:<ID> winner_side:A/B [proof:<attachment>]` | Submits a match result and optional screenshot. Whether proof is mandatory depends on tournament policy. |
| `/match review match_id:<ID> decision:approve/reject/dispute [note:<text>]` | Reviews a pending result. Participants cannot review their own match. |
| `/match cancel match_id:<ID> reason:<text> confirm:True` | Cancels a match but retains its history. |
| `/match reopen match_id:<ID> reason:<text>` | Reopens a rejected/disputed match for new evidence. |
| `/match forfeit match_id:<ID> winner_side:A/B reason:<text>` | Records a manager-approved forfeit/no-show. |
| `/match override match_id:<ID> winner_side:A/B reason:<text> confirm:True` | Overrides a finalized result; reason and confirmation required. |
| `/match status match_id:<ID> status:<ready/in_progress/pending_proof/postponed> [note:<text>]` | Changes the operational match status. |
| `/match request-proof match_id:<ID> reason:<text>` | Rejects current proof and requests new evidence. |
| `/match appeal match_id:<ID> reason:<text>` | Appeals a rejected result. |
| `/match resolve-dispute match_id:<ID> decision:<A/B/reject> reason:<text>` | Resolves a disputed match result. |
| `/match series-appeal match_id:<ID> reason:<text>` | Appeals a finalized best-of series result. |
| `/match series-resolve match_id:<ID> decision:<A/B/reject> reason:<text>` | Resolves a series appeal as an authorized non-participating reviewer/manager. |
| `/match history match_id:<ID>` | Shows match status, scheduling, submission/review notes, and related audit history. |

`winner_side:A` means Side A won; `winner_side:B` means Side B won. Select the side shown in the match channel. Do not guess based on team/player order—check `/match view` or the private match channel.

## Availability commands — `/availability`

| Command | What it does |
|---|---|
| `/availability set tournament_id:<ID> start_at:<ISO time> end_at:<ISO time> [available_window:True/False] [note:<text>]` | Adds an available or unavailable time window. |
| `/availability clear tournament_id:<ID>` | Clears the current user's availability windows for that tournament. |
| `/availability list tournament_id:<ID> [player:<user>]` | Lists saved availability windows. |

Use timezone-aware ISO-8601 timestamps, for example `2026-11-10T18:30:00+05:30`.

## Statistics and export commands

| Command | What it does |
|---|---|
| `/stats player tournament_id:<ID> [player:<user>]` | Shows wins, losses, points, win rate, approved matches, verified power, and evidence status. Defaults to you. |
| `/stats team tournament_id:<ID> team_name:<name>` | Shows team captain, members, and combined member record. |
| `/export results tournament_id:<ID>` | Sends a CSV containing registration rows, match rows, and frozen final placements. |
| `/audit logs [tournament_id:<ID>] [limit:<1–20>]` | Shows recent administrative audit entries. Administrator/Manage Server required. |

## Support commands — `/support`

| Command | What it does |
|---|---|
| `/support create category:<category> description:<text> [tournament_id:<ID>]` | Opens a private support channel. Categories: general, registration, team, match, report, technical, other. |
| `/support claim ticket_id:<ID>` | Staff member claims an open/awaiting-player ticket. |
| `/support awaiting-player ticket_id:<ID> [note:<text>]` | Marks ticket as waiting for the player. |
| `/support resolve ticket_id:<ID> resolution:<text>` | Marks resolved but leaves the channel open. |
| `/support close ticket_id:<ID> resolution:<text>` | Closes the ticket and restricts channel access; history is retained. The ticket owner can close their own ticket. |
| `/support reopen ticket_id:<ID> reason:<text>` | Staff reopens a closed ticket. |
| `/support list [status:<status|all>]` | Lists tickets for support staff. |
| `/support transfer ticket_id:<ID> staff_member:<member>` | Assigns a ticket to another eligible staff member. |
| `/support escalate ticket_id:<ID> reason:<text>` | Escalates to tournament managers/head managers and notifies them by DM where possible. |
| `/support transcript ticket_id:<ID>` | Downloads a text transcript of the ticket channel (up to the latest 1,000 messages in channel history). |

## Server configuration commands — `/config`

| Command | What it does |
|---|---|
| `/config show` | Shows saved category/channels, log channel, and support role. |
| `/config log-channel channel:<channel>` | Sets the audit/log channel. |
| `/config support-role role:<role>` | Sets the support staff role. |

---

## Match lifecycle, proof, appeals, and disputes

A typical match follows this process:

1. **Scheduled:** the match has been created and participants can see its private channel.
2. **Ready / in progress:** staff can update the operational status using `/match status`.
3. **Result submitted:** a participant reports Side A or Side B as the winner and attaches proof if required.
4. **Pending review:** if approval is required, a non-participating authorized manager/reviewer checks the evidence.
5. **Approved:** the result affects standings and can trigger the next round or next game in a series.
6. **Rejected:** the submission is not accepted. The participant may submit new proof after `/match reopen` where permitted, or appeal with `/match appeal`.
7. **Disputed:** the result is held for a manager/reviewer to resolve with `/match resolve-dispute`.

### Who can review?

- Tournament manager roles can manage the tournament and review results, subject to the non-participant rule.
- Reviewer roles are for result-review permissions.
- Head-manager roles are used for escalated disputes and series appeal handling.
- Server administrators/Manage Server users have administrative access in the relevant operations.
- A participant cannot review their own match.

### Forfeit and override

Use `/match forfeit` only when a manager has decided a side forfeited or did not appear. Use `/match override` only to correct an already finalized result. Both require a reason; override also requires confirmation. These actions affect standings and are audit-logged.

### Appeals

- `/match appeal` is for an individual rejected result.
- `/match resolve-dispute` is for an individual disputed result.
- `/match series-appeal` is for a completed best-of series.
- `/match series-resolve` resolves a series appeal. The resolver chooses Side A, Side B, or rejects the appeal and retains the original series winner.

---

## Scheduling and availability

- Configure tournament start/end dates and match duration before scheduling.
- Use `/availability set` to submit available or unavailable windows.
- Use `/availability list` to inspect a player's or the tournament's saved windows.
- Use `/match reschedule` for one match or `/match schedule-all` to schedule unscheduled matches within the configured date window.
- Always include a timezone offset. India example: `+05:30`; UTC example: `Z`.
- Scheduling is an aid; managers should verify the resulting dates/times and communicate changes in the tournament announcement channel.

---

## Standings, prizes, and completion

Useful public-facing commands:

- `/leaderboard` — player standings, optionally filtered by bracket.
- `/team_leaderboard` — team standings.
- `/standings` — wins/losses/points/win rate.
- `/tournament results` — finalized match results.
- `/tournament winners` — frozen final placements after completion.

Before the tournament starts, configure:

1. Prize descriptions with `/tournament prizes`.
2. Optional first/second/third place Discord roles with `/tournament award-roles`.
3. The announcement channel with `/tournament announcement-channel`.

Run `/tournament finish` after the event is complete. The bot freezes the standings snapshot so later match changes do not silently rewrite the published final placements. If unresolved matches remain, review the command response carefully; `force:True` should only be used when the organizer intentionally accepts unresolved matches in the final snapshot.

---

## Support tickets and audit logs

Players can open a private ticket with `/support create`. If a tournament ID is provided, the ticket can include the relevant tournament's non-participating managers/reviewers. Otherwise, the configured server support role and the ticket creator are granted access.

Staff can claim, transfer, mark waiting, resolve, close, reopen, or escalate tickets. Closing a ticket does not delete its channel history. A transcript can be downloaded with `/support transcript` by the ticket owner or authorized staff.

Administrative actions are recorded in MongoDB audit logs. Server administrators can inspect them with `/audit logs`. Match-specific history is available through `/match history`.

---

## Activation system

If this copy of the tournament bot is connected to the SSE Bots Hub activation system:

1. Keep `activation.py` in the same folder as `bot.py`.
2. Set `BOT_ACTIVATION_ID` in `.env` to the activation ID registered for this exact bot in SSE Bots Hub.
3. Set `ACTIVATION_DATABASE=sse_bots_user`.
4. Keep the shared MongoDB URI in `MONGODB_URI`; the activation module selects the activation database separately.
5. The bot must be registered in the Hub with the correct Discord application/bot ID.
6. The actual Discord server owner activates the bot using a valid key. An Administrator role alone is not a substitute for server ownership.

When the server is not activated, the bot restricts tournament commands until activation. Follow the activation instructions supplied with the SSE Bots Hub files. Do not share activation keys.

---

## Environment variables

Create a file named `.env` in the same folder as `bot.py`:

```env
DISCORD_TOKEN=YOUR_DISCORD_BOT_TOKEN
DISCORD_CLIENT_ID=YOUR_NUMERIC_APPLICATION_ID
DISCORD_GUILD_ID=
MONGODB_URI=mongodb+srv://USERNAME:PASSWORD@YOUR_CLUSTER.mongodb.net/sse_tournament_server

# SSE activation (if enabled)
BOT_ACTIVATION_ID=YOUR_TOURNAMENT_ACTIVATION_ID
ACTIVATION_DATABASE=sse_bots_user

# Optional server-wide defaults
LOG_CHANNEL_ID=
DEFAULT_SUPPORT_CATEGORY_ID=
SUPPORT_STAFF_ROLE_ID=

# Optional Android/Pydroid DNS override for MongoDB Atlas SRV records
MONGODB_DNS_NAMESERVERS=1.1.1.1,8.8.8.8

# Optional MongoDB timeouts in milliseconds
MONGODB_SERVER_SELECTION_TIMEOUT_MS=15000
MONGODB_CONNECT_TIMEOUT_MS=10000
MONGODB_SOCKET_TIMEOUT_MS=20000
```

Notes:

- `.env` syntax is `KEY=value`. Do not write `ENV KEY=value` inside the file.
- `DISCORD_CLIENT_ID` is the numeric Application ID from Developer Portal → General Information.
- `DISCORD_TOKEN` is the bot token from Developer Portal → Bot.
- `MONGODB_URI` should point to the tournament database, for example `sse_tournament_server`. The activation module uses `ACTIVATION_DATABASE` separately.
- If your MongoDB password contains special characters, URL-encode them in the connection string.
- Keep `.env` private and out of Git.
- `DISCORD_GUILD_ID` is optional and is used by the bot's command-sync configuration; leave it blank if not using a fixed test guild.

### Start the bot

```bash
python bot.py
```

Keep the process running continuously on your host. If you update the source or `.env`, restart the process.

---

## Troubleshooting

### Slash commands do not appear

- Confirm the bot was invited with both `bot` and `applications.commands` scopes.
- Check that `DISCORD_CLIENT_ID` is the numeric Application ID.
- Confirm the bot is online and has connected successfully.
- Global slash-command updates can take time to appear. For a test server, configure `DISCORD_GUILD_ID` as supported by your deployment setup.
- If activation was completed after the bot started and the slash-command list still shows only activation, restart the bot so it can synchronize the activated command set.

### MongoDB connection or SRV DNS error

- Verify `MONGODB_URI`, database username/password, Atlas network access rules, and cluster status.
- Install `dnspython` for `mongodb+srv://` URIs.
- On Android/Pydroid, try `MONGODB_DNS_NAMESERVERS=1.1.1.1,8.8.8.8`.
- Do not paste your full connection string into a public support channel.

### Bot cannot create match/ticket channels

- Give it Manage Channels, View Channels, Send Messages, and Read Message History.
- Check category permissions and role overwrites.
- Ensure the bot can view the category where the channels should be created.

### Bot cannot assign placement roles

- Give it Manage Roles.
- Move the bot's highest role above all award roles.
- Make sure the target roles are not managed by an integration and are assignable by the bot.

### A match result does not create the next round

- Confirm every match in the current round has an approved/forfeited result.
- Single-elimination, double-elimination, and Swiss progression do not advance a round while a required match is unresolved.
- A draw blocks automatic progression for these formats.
- Round-robin and custom formats require organizers to create the intended fixtures manually.
- Check `/match history` and `/tournament results` for the result status.

### A series does not finish

- Confirm the tournament format is `series`.
- Confirm enough games have finalized to give one side the required majority.
- For Bo3, a side needs 2 wins; for Bo5, 3 wins; for Bo7, 4 wins.
- A pending, rejected, or disputed game does not count as an approved series game.

### User cannot register or join a team

- Confirm registration is open.
- Check the tournament ID.
- In team mode, ensure the user is registered before team membership actions.
- Check team size, invitation-only settings, and whether the team is locked.

### Activation is unavailable

- Confirm `activation.py` is beside `bot.py`.
- Confirm `BOT_ACTIVATION_ID` exactly matches the Hub's registered activation ID.
- Confirm the Hub has the correct Discord bot/application ID.
- Confirm `ACTIVATION_DATABASE=sse_bots_user` and that the MongoDB user can access that database.
- Only the actual Discord server owner can activate the bot.

---

## Data and operational notes

The bot uses MongoDB collections for games, tournaments, registrations, teams, matches, support tickets, audit logs, server configuration, series records, and availability. Keep regular backups of the database. Deleting a tournament is destructive to its database records; use archive/finish when you want to preserve the tournament record. Deleting related Discord channels is optional during tournament deletion, and disputed matches block deletion.

The bot can only perform actions allowed by Discord permissions and the configured tournament roles. Organizers should publish the format, scoring, tie-breakers, bracket ranges, evidence requirements, deadlines, and appeal rules before opening registration.

---

**SSE Tournament Bot**  
Part of the SSE World / SSE Universe bot ecosystem.
