# 64ers

A simple Vue.js application that allows you to encode files to Base64 and decode Base64 strings back into downloadable files using automatic mime detection.

## Features

- **Encode mode:** Drag and drop or select a file to convert it to a Base64 string.
- **Decode mode:** Paste a Base64 string to generate and download the original file.
- **Mime-type detection:** Automatically guesses file type by inspecting the Base64 signature. If unknown, prompts for file extension.

## Prerequisites

- [Node.js](https://nodejs.org/) (v14 or later)
- npm (bundled with Node.js)

## Setup & Run

1. **Clone the repository**
   ```bash
   git clone https://github.com/badmuriss/64ers.git
   cd 64ers
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Start the development server**
   ```bash
   npm run serve
   ```
   Open [http://localhost:5173](http://localhost:5173) in your browser.

4. **Build for production**
   ```bash
   npm run build
   ```
   The compiled files will be in the `dist` folder.

## Usage

- Toggle between **Encode** and **Decode** modes using the switch at the top.
- In **Encode** mode, drag & drop a file or click to select. The Base64 result appears in the output area.
- In **Decode** mode, paste the Base64 text. If the app cannot detect the file type, you’ll be prompted to enter the file extension (e.g., `png`, `pdf`).
- Click **Download** to save the decoded file.



