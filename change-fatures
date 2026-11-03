# SSE Tournament Bot — Changelog, Features & Roadmap

**Document version:** 1.0  
**Last updated:** 3 November 2026  
**Project:** SSE Tournament Bot  
**Status labels:** `Implemented` = present in the inspected bot source; `Planned` = proposal for a future release and not claimed to exist today.

> This document is based on the current `bot.py`, `help_menu.py`, and `activation.py` files available during preparation. Future ideas are deliberately separated from implemented functionality.

---

## 1. Product overview

SSE Tournament Bot is a Discord tournament-management bot. It provides server activation controls, tournament setup and lifecycle management, participant registration, teams, brackets, matchmaking, match reporting and review, standings, support tickets, configuration, and audit-related tools.

## 2. Current features

### Server activation and setup

- Activate the bot for a Discord server using an activation key.
- Temporarily deactivate the bot for a server.
- Reactivate a previously deactivated server.
- Permanently remove a server's activation record.
- Create the base tournament channels with `/setup`.
- Remove bot-created tournament, match, and ticket channels with `/cleanup`.

### Tournament management

- Create, list, edit, archive, delete, and clone tournaments.
- Configure tournament maps, schedule, scoring, and result policies before start.
- Open, close, and reopen registration.
- Start, pause, resume, finish, and cancel tournaments.
- Set an announcement channel.
- Configure first-, second-, and third-place prize descriptions.
- Configure manager, reviewer, and head-manager roles.
- Configure award roles for final placements.
- View finalized results and frozen winners.

### Registration and squad-power review

- Register for a tournament and submit squad-power evidence.
- Withdraw from a tournament.
- Move participants between brackets.
- Remove or disqualify participants.
- Review squad-power submissions and approve or reject them.
- Correct a verified average squad-power value with an audit reason.

### Teams

- Create, list, edit, and delete teams.
- Join or leave a team.
- Invite players and accept invitations.
- Kick team members.
- Transfer team captaincy.
- Substitute one team member for another.
- Disqualify a team.
- View team standings and team statistics.

### Brackets

- Add, list, edit, reorder, and delete average squad-power brackets.
- Move a participant to a manager-selected bracket.

### Matchmaking and matches

- Generate initial match pairings and private match channels.
- Preview proposed pairings without creating channels.
- Regenerate unstarted matches after confirmation.
- Manually create matches for custom formats or rematches.
- View match details and status.
- Edit scheduled match maps and reporting deadlines.
- Reschedule matches or automatically schedule unscheduled matches.
- Repair a missing private match channel.
- Submit match results with screenshot proof.
- Review results by approving, rejecting, or disputing submissions.
- Cancel or reopen matches.
- Record a manager-approved forfeit or no-show.
- Override a finalized result with a reason.
- Set match status.
- Request new proof, appeal a rejected result, and resolve match or series disputes.
- View match history.

### Standings, availability, and statistics

- View tournament leaderboard and standings.
- View team tournament leaderboard.
- Add, clear, and list participant availability windows.
- View player and team statistics.

### Support and administration

- Open private support tickets in configured categories.
- Claim, transfer, and escalate tickets.
- Mark tickets as awaiting a player.
- Resolve, close, or reopen tickets.
- List tickets and download transcripts.
- Show bot channel/role configuration.
- Set the audit/log channel and support role.
- View recent audit log entries.
- Export tournament results to CSV.
- Post tournament announcements.

---

## 3. Current slash-command reference

The command names below were collected from the command registrations in the inspected `bot.py`.

### Game management
`/game create`, `/game delete`, `/game edit`, `/game list`

### Tournament management
`/tournament add-head-manager-role`, `/tournament add-manager-role`, `/tournament add-reviewer-role`, `/tournament announcement-channel`, `/tournament archive`, `/tournament award-roles`, `/tournament cancel`, `/tournament clone`, `/tournament configure`, `/tournament create`, `/tournament delete`, `/tournament edit`, `/tournament finish`, `/tournament list`, `/tournament pause`, `/tournament prizes`, `/tournament remove-head-manager-role`, `/tournament remove-manager-role`, `/tournament remove-reviewer-role`, `/tournament results`, `/tournament resume`, `/tournament start`, `/tournament winners`

### Registration
`/registration close`, `/registration disqualify-player`, `/registration open`, `/registration remove-player`, `/registration reopen`

### Bracket management
`/bracket add`, `/bracket delete`, `/bracket edit`, `/bracket list`, `/bracket move-player`, `/bracket reorder`

### Team management
`/team accept`, `/team create`, `/team delete`, `/team disqualify`, `/team edit`, `/team invite`, `/team join`, `/team kick`, `/team leave`, `/team list`, `/team substitute`, `/team transfer-captain`

### Match management
`/match appeal`, `/match cancel`, `/match create`, `/match edit`, `/match forfeit`, `/match history`, `/match override`, `/match reopen`, `/match repair-channel`, `/match request-proof`, `/match reschedule`, `/match resolve-dispute`, `/match review`, `/match schedule-all`, `/match series-appeal`, `/match series-resolve`, `/match status`, `/match submit`, `/match view`

### Player availability
`/availability clear`, `/availability list`, `/availability set`

### Statistics
`/stats player`, `/stats team`

### Support tickets
`/support awaiting-player`, `/support claim`, `/support close`, `/support create`, `/support escalate`, `/support list`, `/support reopen`, `/support resolve`, `/support transcript`, `/support transfer`

### Configuration
`/config log-channel`, `/config show`, `/config support-role`

### Audit logs
`/audit logs`

### Exports
`/export results`

### Standalone commands
`/activate`, `/announce`, `/cleanup`, `/deactivate`, `/leaderboard`, `/matchmaking`, `/matchmaking_preview`, `/matchmaking_regenerate`, `/reactivate`, `/register`, `/remove`, `/setup`, `/standings`, `/team_leaderboard`, `/unregister`


**Additional note:** The bot also has a separate interactive `/help` menu module. Its categories and action menus are intended to display the registered commands and their parameter descriptions.

---

## 4. Changelog

### 3 November 2026 — Help, command descriptions, and server controls

- Added an interactive help-center design with category and action dropdowns.
- Added a separate `help_menu.py` module for the help interface.
- Updated command-option descriptions so missing descriptions and literal `...` placeholders receive readable help text.
- Added/updated server lifecycle commands: `/activate`, `/deactivate`, `/reactivate`, and `/remove`.
- Kept `/cleanup` as the separate command for deleting bot-created channels.
- Reduced routine console output while retaining the `SSE Tournament is ready.` startup message and error visibility.
- Improved the help-menu handling for empty command-option lists.

### Existing tournament-management capabilities

The current source also includes tournament lifecycle controls, registration, team and bracket management, matchmaking, match-result workflows, standings, support tickets, configuration, audit logs, and CSV result export. See the feature sections above for the command-level inventory.

---

## 5. Future roadmap — proposed features

**Everything in this section is a suggestion for future development. These features are not represented as implemented in the current bot source.**

### Priority 1 — Tournament experience

- [ ] **Public tournament dashboard:** A web dashboard showing live brackets, match status, standings, and upcoming fixtures.
- [ ] **Live bracket visualization:** Automatically update a visual single-elimination or double-elimination bracket as results are approved.
- [ ] **Tournament check-in:** Let registered players confirm attendance before the tournament begins; automatically flag no-shows.
- [ ] **Waitlist system:** Automatically move waitlisted participants into open slots when someone withdraws.
- [ ] **Automatic round progression:** Generate the next round after all required matches in the current round are finalized.
- [ ] **Tournament templates:** Save reusable configurations for recurring tournament formats.

### Priority 2 — Match integrity and fairness

- [ ] **Evidence gallery:** Store and display all match screenshots and submitted evidence in one match record.
- [ ] **Dispute timeline:** Show every submission, review, appeal, and manager decision with timestamps.
- [ ] **Anti-smurf / duplicate-account checks:** Flag suspicious repeat registrations for manual review.
- [ ] **Automated no-show handling:** Notify both sides before a deadline and create a review task when a player does not appear.
- [ ] **Conflict-of-interest protection:** Prevent a reviewer from reviewing a match involving their own team.
- [ ] **Review SLA reminders:** Remind assigned reviewers when a result or dispute has been waiting too long.

### Priority 3 — Player and team experience

- [ ] **Player profile cards:** Show tournament history, wins, losses, win rate, awards, and team history.
- [ ] **Player ELO / rating system:** Calculate a rating from finalized match results, with configurable rating rules.
- [ ] **Team chemistry and roster history:** Track roster changes and historical team performance.
- [ ] **Achievement badges:** Award badges for milestones such as first win, tournament champion, or a winning streak.
- [ ] **Player notification preferences:** Let users choose which reminders and tournament notifications they receive.
- [ ] **Team recruitment board:** Allow captains to publish open team slots and players to apply.

### Priority 4 — Automation and integrations

- [ ] **Scheduled tournament reminders:** Send configurable reminders for registration deadlines, match times, and result deadlines.
- [ ] **Calendar integration:** Export fixtures to Google Calendar or standard `.ics` calendars.
- [ ] **Streaming integration:** Add Twitch/YouTube stream links to matches and tournament announcements.
- [ ] **Automatic highlight posts:** Publish match winners and tournament milestones to configured channels.
- [ ] **External game-stat integration:** Import supported game statistics where a reliable official API is available.
- [ ] **Webhook/API access:** Allow an external website or dashboard to read tournament data securely.

### Priority 5 — Administration and reliability

- [ ] **Web-based server control panel:** Manage tournament settings, roles, channels, and activation from a dashboard.
- [ ] **Database backup and restore:** Create scheduled backups and provide a tested restore workflow.
- [ ] **Audit export:** Export administrative actions and review history for a selected date range.
- [ ] **Permission simulator:** Preview which Discord roles can run or review each command before applying changes.
- [ ] **Health monitoring:** Report database connectivity, command-sync status, and background-task health.
- [ ] **Multi-language support:** Add translated command help and bot messages.
- [ ] **Feature toggles:** Let server administrators enable or disable optional modules without removing the bot.
- [ ] **Rate limits and abuse protection:** Add configurable cooldowns and safeguards for high-impact commands.

---

## 6. Suggested release plan

| Release | Suggested scope |
|---|---|
| v1.1 | Tournament check-in, waitlist, and scheduled reminders |
| v1.2 | Visual bracket and automatic round progression |
| v1.3 | Player profiles, ELO, and achievements |
| v1.4 | Evidence gallery and improved dispute timeline |
| v2.0 | Web dashboard, API, and database backup/restore |

These are planning suggestions, not committed release dates.

---

## 7. Contribution and feature tracking

When a planned feature is implemented:

1. Move it from the Future roadmap into Current features.
2. Add the release date and a short description under Changelog.
3. Update the current slash-command reference if commands were added or changed.
4. Test permission checks, error handling, and database behavior.
5. Keep the documentation synchronized with the actual bot source.

---

*Maintained for SSE Tournament Bot. Update this document whenever a feature is shipped, changed, or removed.*
