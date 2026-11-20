
# Palia Mining Party Calculator

This is a Vue 3 application I created with the help of AI for my web development class.

## About the App

I organize mining parties in Palia and wanted an easier way to calculate everyone's scores.

The calculator lets users choose between three mining locations: Bahari Bay, Elderwood, and Royal Highlands. Each location has its own gems and scoring rules.

## Features

- Choose a mining location
- Enter quantities of collected gems
- Calculate double points for star-quality gems
- Add bonus points for Kitsuu plushies
- Customize gem point values
- Save individual player scores
- Automatically calculate team totals
- View team and individual leaderboards
- Export scores to a CSV file
- Copy a readable summary of the player name, team, mining location, and total score
- Save scores in browser storage

## Mobile Layout

The mobile layout gives gemstone names and point values more room. Normal and Star counters have clear labels and larger input fields, and buttons are easier to tap on a phone-sized screen.

## Technologies Used

- Vue 3
- JavaScript
- Vite
- HTML and CSS
- AI-assisted development

## Custom AI Instructions

I used custom AI instructions in `.github/copilot-instructions.md` to guide development. They asked for beginner-friendly Vue 3 and JavaScript, preserving the existing features and dark theme, keeping the app responsive, and building the project to check for errors.

## Testing Results

| Test | Result |
| --- | --- |
| Copy Results | Passed — Correct player, team, location, and score copied. |
| Mobile Layout | Passed — Application displays correctly on a phone-sized screen. |
| Save Player Score | Passed — Player and team appear correctly on the leaderboards with 7 points. |

## Running the App

1. Download or clone the repository.
2. Open the project folder in VS Code.
3. Run `npm install`.
4. Run `npm run dev`.
5. Open the local URL displayed in the terminal.

## Netlify Deployment

Deployment URL: https://palia-mining-calculator.netlify.app/

## Possible Future Improvements

- Add gemstone images.
- Make it easier to manage players and teams during mining parties.
