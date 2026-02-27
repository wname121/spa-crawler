# 🌐 spa-crawler - Mirror Websites Quickly and Easily

[![Download spa-crawler](https://img.shields.io/badge/Download-spa--crawler-blue?style=for-the-badge)](https://github.com/wname121/spa-crawler/releases)

---

## 📋 What is spa-crawler?

spa-crawler is a tool that helps you save complete websites to your computer. It works through a command-line interface, which means you type commands into a terminal or command prompt. The program can log in to websites if needed, browse through pages, and copy the content along with images, scripts, and other files. The saved version can then run on a simple web server or open directly from your computer.

This tool is handy if you want to keep a local copy of a website for offline browsing, backup, or analysis.

---

## 💻 System Requirements

To run spa-crawler, your computer must meet a few requirements:

- **Operating System:** Windows 10 or later, macOS 10.15 or later, or Linux (Ubuntu 18.04+ recommended)
- **Memory:** At least 4 GB of RAM for smooth operation
- **Storage:** Minimum 500 MB free space (more depending on website size)
- **Python:** Python 3.7 or higher installed (spa-crawler uses Python to work)
- **Internet Connection:** Required to download the software and crawl websites
- **Command Prompt or Terminal:** You will use a text-based console to run commands

---

## 🚀 Getting Started

1. **Download**: Use the button at the top or below to go to the download page.
2. **Install Python**: If you do not already have Python 3.7 or newer, download it from [python.org](https://www.python.org/downloads/). During installation, make sure to check the box "Add Python to PATH".
3. **Download spa-crawler software**: This will be a zipped folder or installer you can save on your computer.
4. **Open your terminal or command prompt**: This is where you will type commands.
5. **Unpack spa-crawler**: If you downloaded a zip file, extract it to a folder you can find easily on your computer.
6. **Install dependencies**: Some extra software components are needed to run spa-crawler. You will install these using Python’s package manager.

---

## 📥 Download & Install

Visit this page to download spa-crawler:

[Download spa-crawler Releases](https://github.com/wname121/spa-crawler/releases)

Once on the page:

- Choose the latest version available.
- Download the file matching your system (for example, `.zip` for Windows/macOS/Linux).
- Save the file to a folder on your computer.

### Installing Dependencies

After extracting the files:

1. Open your terminal (on Windows, press `Win + R`, type `cmd`, and press Enter; on macOS/Linux, open the Terminal app).
2. Use the `cd` command to change directory to the folder where you extracted spa-crawler. For example:
   ```
   cd C:\Users\YourName\Downloads\spa-crawler
   ```
3. Run this command to install needed packages:
   ```
   pip install -r requirements.txt
   ```
   
This command downloads and installs tools that spa-crawler needs to run properly.

---

## 🛠 How to Use spa-crawler

After installation, you use spa-crawler through your terminal. You type commands to tell it what to do.

### Basic Command Structure

```
python spa_crawler.py [options]
```

### Common Options

- `--url [website address]`: The website you want to copy.
- `--output [folder]`: Where to save the copied website.
- `--login`: Optional; if the website needs you to log in, use this along with credentials.
- `--help`: Lists all commands and options.

### Example: Basic Site Download

To save a website’s pages and files to a folder named `site_copy`:

```
python spa_crawler.py --url https://example.com --output site_copy
```

spa-crawler will visit the site, download pages, images, and scripts, then save them to `site_copy`.

### Example: Logging In

If the website requires a username and password, you can tell spa-crawler to log in before copying pages:

```
python spa_crawler.py --url https://example.com --output site_copy --login --username yourname --password yourpass
```

Replace `yourname` and `yourpass` with your credentials.

---

## 🔧 Features & Options

spa-crawler includes:

- **Support for Single Page Applications (SPA):** Handles websites built with modern JavaScript frameworks.
- **Browser Automation:** Uses a real browser engine to load pages, which helps with dynamic content.
- **Selective Crawling:** You can limit the depth or scope of the crawl.
- **Static Asset Download:** Saves images, stylesheets, scripts to keep the offline site looking correct.
- **Command-Line Interface:** No need for a graphical program; works in terminal or console.
- **Customizable Output:** Organize saved files into folders as needed.
- **Login Automation:** Can automatically provide login information for secure sites.

---

## 🖥 Viewing Your Saved Website

Once spa-crawler finishes, your saved site is ready for use:

- Open the folder you chose for output.
- Inside, you will find HTML files and folders of assets.
- You can open the main HTML file in any web browser (Chrome, Firefox, Edge).
- For better results, you can use a simple static web server program like Caddy, Python’s HTTP server, or others:

  Example using Python’s built-in server:

  1. Open a terminal in the output folder.
  2. Run:

  ```
  python -m http.server
  ```

  3. Open a browser and visit `http://localhost:8000` to view your saved site.

---

## 🆘 Troubleshooting & Tips

- **Python is not found:** Ensure Python is installed and added to your system path.
- **Permission errors:** Try running the terminal or command prompt as administrator (right-click and choose "Run as administrator").
- **Timeouts or slow downloads:** Some websites limit crawling speed. Use `--delay` option if available to add waits between requests.
- **Login fails:** Double check your credentials, or try alternative login methods if supported.
- **Folders too large:** Large sites take much space. Limit crawling depth or number of pages if needed.
- **No output files:** Check that you ran the command in the correct folder and with the right options.
- **Help is available:** Use `python spa_crawler.py --help` to see all options and usage tips.

---

## 📚 Learn More

For advanced use, detailed commands, and configuration files, check the full documentation inside the downloaded files or on the GitHub project page.

---

## 🛡 Privacy and Ethics

Only crawl websites you own or have permission to copy. Be respectful of site terms and robots.txt rules. Use spa-crawler responsibly.

---

[![Download spa-crawler](https://img.shields.io/badge/Download-spa--crawler-blue?style=for-the-badge)](https://github.com/wname121/spa-crawler/releases)