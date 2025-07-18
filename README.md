# Gemini YouTube Page Summarizer Extension

This browser extension adds a button to **YouTube** pages (both desktop and mobile). Clicking the button prepares a summarization prompt (including the current YouTube page's URL), stores it temporarily, and opens Gemini in a new tab. A companion script on the Gemini page then attempts to automatically fill this prompt into the input field.

## Core Features

- Adds a "✨ Summarize with Gemini" button to YouTube pages (`youtube.com` and `m.youtube.com`).
- When clicked, stores a prompt (`Please summarize this YouTube page: [URL]`) for the Gemini page.
- Opens `https://gemini.google.com/app` in a new tab.
- Automatically fills the prompt into Gemini's input field when it becomes available.
- Handles cases where the input field is disabled on initial page load.
- Prevents duplicate form submissions.
- Provides detailed logging for troubleshooting.

## Security and Best Practices

We prioritize security and user privacy. For details on the best practices implemented in this extension, please see [BEST_PRACTICES.md](BEST_PRACTICES.md).

## Privacy Policy

This extension handles your data responsibly. Please review our [Privacy Policy](PRIVACY.md) to understand what information is used and how it is handled.

## Disclaimer

This is an independent extension and is not affiliated with, endorsed by, or sponsored by Google, YouTube, or Gemini. All product names, logos, and brands are property of their respective owners.

Please be aware that using the Gemini service may be subject to Google's terms and policies. If you have billing enabled for Google Cloud services, charges for Gemini API usage may apply. This extension does not control or manage these charges.

For further details regarding data handling, please refer to the [Privacy Policy](PRIVACY.md).

## Installation

### For Chrome:

1.  Download or clone this repository to your local machine.
2.  Open Chrome and navigate to `chrome://extensions`.
3.  Enable "Developer mode" (usually a toggle in the top right).
4.  Click "Load unpacked".
5.  Select the `dist/chrome` directory from your local repository (this directory is created after you build the extension, see "Building the Extension" below).

### For Firefox:

1.  Download or clone this repository to your local machine.
2.  Open Firefox and navigate to `about:debugging#/runtime/this-firefox`.
3.  Click "Load Temporary Add-on...".
4.  Navigate to the `dist/firefox` directory in your local repository (this directory is created after you build the extension) and select the `manifest.json` file.

## Usage

1.  Once installed, navigate to a YouTube video page (e.g., `https://www.youtube.com/watch?v=...` or `https://m.youtube.com/watch?v=...`).
2.  You should see a "✨ Summarize with Gemini" button appear on the page (its exact location might vary depending on YouTube's layout).
3.  Click the "✨ Summarize with Gemini" button.
4.  A new tab will open to `https://gemini.google.com/app`.
5.  The extension will attempt to automatically paste a prompt like "Please summarize this YouTube page: [URL_OF_THE_VIDEO]" into Gemini's input field.
6.  If auto-fill is successful, you can add to the prompt or send it to Gemini.
7.  If auto-fill fails (e.g., due to changes in Gemini's page structure or if you navigate away too quickly), the extension will show an alert asking you to return to YouTube and click the summarize button again. The alert message will be similar to: "Gemini Summarizer: Could not automatically fill the prompt. Please return to YouTube and click the summarize button again."
8.  You can manage extension options (like enabling verbose logging for development) by clicking the extension icon in your browser's toolbar and selecting "Options" (if available), or by finding the extension in your browser's extension management page and accessing its options from there.

## Building the Extension

To set up the development environment and build the extension:

### Prerequisites:

-   [Node.js](https://nodejs.org/) (which includes npm) installed on your system.

### Build Steps:

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/bhare1985/youtube-summarizer-extension.git
    cd youtube-summarizer-extension
    ```
2.  **Install dependencies:**
    ```bash
    npm install
    ```
3.  **Build the extension:**
    ```bash
    npm run build
    ```
    This command compiles the necessary files and places them into the `dist/chrome` and `dist/firefox` directories. These are the directories you'll use to load the unpacked extension into your browsers.

4.  **Package the extension (optional, for distribution):**
    ```bash
    npm run package
    ```
    This command first runs the build, then creates `chrome.zip` and `firefox.zip` files in the `dist/` directory. These zip files are suitable for distribution or uploading to extension stores.

## Development

This extension is built using Manifest V3.

-   `manifest.json`: Extension configuration, permissions, content script declarations, and browser-specific settings.
-   `content.js`: Injects the "Summarize with Gemini" button onto YouTube pages and handles the initial click interaction (opening Gemini and storing the URL).
-   `gemini_filler.js`: Runs on `gemini.google.com/app` to attempt to fill the prompt (containing the YouTube URL) from `chrome.storage.local`.
-   `logging.js`: Provides shared logging utilities (`devLog`, `devWarn`, `devError`) for content scripts, controllable via a setting in `options.html`.
-   `options.html` & `options.js`: Provide an options page for the extension (e.g., to enable verbose logging).
-   `style.css`: Styles for the injected button and other UI elements.
-   `images/`: Contains the extension icons (icon16.png, icon48.png, icon128.png).

-   `tests/`: Contains integration tests.
    -   `tests/integration/youtube_gemini_flow.spec.js`: Main integration test script using Playwright.
    -   **Note**: The integration tests rely on CSS selectors within `content.js` (for YouTube button placement) and `gemini_filler.js` (for the Gemini input field). These selectors might need updates if YouTube or Gemini changes their page structure.

### Running Tests

This project uses Playwright for integration testing. You can run the tests using the following npm scripts:

-   **Run all main integration tests:**
    ```bash
    npm test
    ```
    This executes the tests defined in `tests/integration/youtube_gemini_flow.spec.js`.

-   **Run a quicker, focused test (e.g., button presence):**
    ```bash
    npm run test:quick
    ```
    This executes tests from `tests/integration/youtube_button_test.spec.js`.

-   **Run tests in headed mode (visible browser for debugging):**
    ```bash
    npm run test:headed
    ```

-   **Generate test code with Playwright Codegen:**
    ```bash
    npm run codegen
    ```
    This opens the Playwright test generator, which can help you create new tests by recording your interactions.

Make sure you have run `npm install` to install Playwright and other testing dependencies before running the tests.

## Key Technical Points & Challenges

-   **Dynamic Content on YouTube & Gemini**: Both sites are Single Page Applications (SPAs). Button injection and prompt filling rely on DOM event listeners (like `yt-navigate-finish`, `load`), polling mechanisms (`setInterval`), careful timing, and robust CSS selector choice.
-   **Selector Stability**: CSS selectors for YouTube and Gemini elements can change, potentially requiring updates to `content.js` and `gemini_filler.js`.
-   **Playwright with Manifest V3 Extensions**: Testing Manifest V3 extensions, especially those with specific launch requirements (like loading unpacked extensions), using Playwright in headless mode requires careful configuration. The `playwright.config.js` and test setup demonstrate a working approach using arguments like `--disable-extensions-except`, `--load-extension`, and `--headless=chromium`.
*   **Cross-Browser Considerations (Firefox ID)**: Core storage APIs (`chrome.storage.local`) are generally consistent across browsers thanks to the WebExtensions standard. However, for proper identification and functionality on Firefox (e.g., updates via AMO), the Firefox-specific `browser_specific_settings.gecko.id` must be defined in `manifest.json`.

## Version History

### v0.0.3
- Improved input field detection and handling
- Added MutationObserver to detect when the input field becomes enabled
- Added protection against duplicate form submissions
- Enhanced error handling and logging
- Improved reliability when the Gemini interface is slow to load
- Added visual feedback during form submission

### v0.0.2
- Initial release with basic functionality

## Planned Features / Improvements

-   User-configurable prompt templates.
-   More robust and adaptive button placement logic for YouTube.
-   Improved visual feedback for the user.
-   Allow summarization of videos linked from pages other than YouTube itself (e.g., embedded videos, if feasible and permitted).

## Contributing

Contributions are welcome! Please feel free to submit a pull request or open an issue for any bugs, feature requests, or suggestions.

## License

This project is licensed under the MIT License.
