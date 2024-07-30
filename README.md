# 🔐 TeleOTP
[![Build Telegram bot image](https://github.com/UselessStudio/TeleOTP/actions/workflows/bot.yml/badge.svg)](https://github.com/UselessStudio/TeleOTP/actions/workflows/bot.yml)
[![Telegram channel](https://img.shields.io/badge/Telegram-channel-blue)](https://t.me/teleotpapp)
[![Plausible analytics](https://img.shields.io/badge/Plausible-analytics-blue)](https://analytics.gesti.tech/teleotp.gesti.tech)

Telegram Mini App that allows you to generate one-time 2FA passwords inside Telegram.

[➡️ **OPEN**]([TeleOTP](http://t.me/TeleOTPAppBot))

## ✨ Features

* ✅ **Universal:** TeleOTP implements [TOTP](https://en.wikipedia.org/wiki/Time-based_one-time_password) (Time-Based One-Time Password Algorithm) which is used by most services.
* 👌 **Convenient:** Accounts are safely stored in your Telegram cloud storage, 
so you can access them anywhere you can use Telegram. 
* 🔒 **Secure:** All accounts are encrypted using AES. 
That means even if your Telegram account is breached, 
the attacker won't have access to your tokens without the encryption password. 
* 🥰 **User-friendly:** TeleOTP is designed to look like Telegram and follows your color theme.
* 🤗 **Open:** TeleOTP supports account migration to and from Google Authenticator. 
You can switch the platforms at any time without any hassle! 

## Table of contents
<!-- TOC -->
* [🔐 TeleOTP](#-teleotp)
  * [✨ Features](#-features)
  * [Table of contents](#table-of-contents)
* [⚙️ Setup guide](#-setup-guide)
  * [📱 Mini App](#-mini-app)
    * [Installing the dependencies](#installing-the-dependencies)
    * [Configuring the environment](#configuring-the-environment)
    * [Starting the development server](#starting-the-development-server)
    * [Building the app](#building-the-app)
    * [🔁 CI/CD](#-cicd)
  * [💬 Bot](#-bot)
    * [Starting bot](#starting-bot)
    * [Environment variables](#environment-variables)
    * [Running in Docker](#running-in-docker)
    * [🔁 CI/CD](#-cicd-1)
* [💻 Structure](#-structure)
  * [🛣️ Routing](#-routing)
  * [🤖 Data and business logic](#-data-and-business-logic)
    * [✈️ Migration](#-migration)
      * [Caveats](#caveats)
    * [🤗 Icons and colors](#-icons-and-colors)
* [👋 Acknowledgements](#-acknowledgements)
  * [🖌️ Content](#-content)
  * [📚 Libraries used](#-libraries-used)
<!-- TOC -->

# ⚙️ Setup guide

## 📱 Mini App

TeleOTP is made with **React**, **Typescript**, and **[Material UI](https://mui.com/material-ui/)**. 
**Vite** frontend tooling is used for rapid development and easy deployment.

### Installing the dependencies

To begin working with the project, 
you should install the dependencies by running this command:

```shell
npm install
```

### Configuring the environment

Before starting the server or building the app, make sure that 
your project directory has a file named `.env`. 
It should follow the `.env.example` file structure.
You could also set these variables directly when running the app.

The app uses following environment variables:

* `VITE_BOT_USERNAME` - This value contains the bot username. 
It is used to send export requests to the bot. 
_Example: `VITE_BOT_USERNAME=TeleOTPAppBot`_
* `VITE_PLAUSIBLE_DOMAIN` - The domain for Plausible analytics.
* `VITE_PLAUSIBLE_API_HOST` - API host to report Plausible analytics to.
* `VITE_CHANNEL_LINK` - Link to the news channel.

### Starting the development server
To start the development server with hot reload, run:

```shell
npm run dev
```

After that, the server will be accessible on http://localhost:5173/

> [!NOTE]
> If you want the app to be accessible on your local network, you should add `--host` argument to the command

```shell
npm run dev -- --host
```

### Building the app

```shell
npm run build
```
After a successful build, app bundle will be available in `./dist`.

### 🔁 CI/CD
The app is built and deployed automatically using **Cloudflare Pages**.

## 💬 Bot

TeleOTP uses a helper bot to send user a link to the app and assist with account migration.
The bot is written in **Python** using [Python Telegram Bot library](https://github.com/python-telegram-bot/python-telegram-bot).

### Starting bot

To start the bot, you have to run the `main.py` script with environment variables.
```shell
python main.py
```

### Environment variables

* `TOKEN` - Telegram bot token provided by @BotFather
* `TG_APP` - A link to the Mini App in Telegram (e.g. https://t.me/TeleOTPAppBot/app)
* `WEBAPP_URL` - Deployed Mini App URL (e.g. https://uselessstudio.github.io/TeleOTP)

> [!NOTE]
> Make sure that `WEBAPP_URL` doesn't end with a `/`! 
> It is added automatically by the bot.

### Running in Docker

We recommend running the bot inside the Docker container. 
The latest image is available at `ghcr.io/uselessstudio/teleotp-bot:main`.

Example `docker-compose.yml` file:

```yaml
services:
  bot:
    image: ghcr.io/uselessstudio/teleotp-bot:main
    restart: unless-stopped
    environment:
      - TG_APP=https://t.me/TeleOTPAppBot/app
      - WEBAPP_URL=https://uselessstudio.github.io/TeleOTP
      - TOKEN=<insert your token>
```

And running is as simple as:
```shell
docker compose up
```

### 🔁 CI/CD
GitHub Actions is used to automate the building of the bot container.
The workflow is defined in the [`bot.yml` file](.github/workflows/bot.yml)
and ran on every push to `main`. After a successful build, 
the container is published in the GitHub Container Registry.  


# 💻 Structure

## 🛣️ Routing

TeleOTP uses React Router to switch between pages. 
Routes are specified in the `main.tsx` file.

* Route implemented in `Root.tsx` is responsible for showing "required" screens:
  * `PasswordSetup.tsx` is a screen which is showed when no password is created.
  Alternatively, this screen is shown when user clicked a button to change the password.
  It displays a prompt to create a new password which is used to encrypt the accounts.
  * `Decrypt.tsx` is a screen which shows a password prompt to decrypt stored accounts.
  By default, it is shown only once on a new device. 
  Later, the password retrieved from `localStorage` (if not disabled in the settings).
* `Accounts.tsx` is the main screen, which shows the generated password and a list of accounts that user has.
* `NewAccount.tsx` is a screen, which prompts user to open the QR-code scanner. 
Otherwise, user could press a button to enter an account manually, which would redirect them to `ManualAccount.tsx`
* `ManualAccount.tsx` prompts user to enter a secret for an OTP account.
* `CreateAccount.tsx` is a final step in the account creation flow. User is redirected here after scanning a QR-code 
or after manually providing a secret. 
This screen allows user to change issuer and label and to select an icon with a color for the account.
* `EditAccount.tsx` is a screen similar to `CreateAccount.tsx` which allows 
user to edit or delete an account. 
* `Settings.tsx` is a menu screen with a few options. User can delete all accounts, 
encrypt them, change the password, or set preferences.
* `ExportAccounts.tsx` is a page that handles the export logic. 
This page could be opened only by pressing the keyboard button in the chat. 
It sends the exported accounts back to the bot.
* `ResetAccounts.tsx` is a page that verifies that the user wants to delete all accounts and reset the password. 
This page can be accessed through the settings, or by typing in the password incorrectly when decrypting.

## 🤖 Data and business logic

### ✈️ Migration

TeleOTP implements the `otpauth-migration` URI standard. 
During the migration, accounts are serialized using Protocol Buffers 
and sent to the bot via `sendData` method. Bot then generates the QR-code and a link, 
that can be used for migrating to another instance of TeleOTP.

#### Caveats

The length of the data that can be sent using `sendData` is limited to 4096 bytes. 
So the quantity of the accounts that could be migrated from TeleOTP is limited. 
Although, this number theoretically is pretty big, it is still smaller than 
the amount of the accounts that can be stored in CloudStorage (around 1000).   

### 🤗 Icons and colors

All generic icons and colors for accounts are defined in the `globals.tsx` file.
Available icons are exported in the `icons` const, colors are available in the `colors` const.
`Icon` and `Color` types are provided for checking the validity of the corresponding item.

# 👋 Acknowledgements

## 🖌️ Content

* Emoji animations from [Telegram stickers](https://t.me/addstickers/AnimatedEmojies).
* [Duck stickers](https://t.me/addstickers/UtyaDuck)
* Brand icons from [Simple Icons](https://simpleicons.org/)

## 📚 Libraries used

* [@twa-dev/types](https://github.com/twa-dev/types) - Typescript types for Telegram Mini App SDK 
* [OTPAuth](https://www.npmjs.com/package/otpauth) - generating TOTP codes
* [nanoid](https://www.npmjs.com/package/nanoid) - generating unique ids for accounts
* [lottie-react](https://www.npmjs.com/package/lottie-react) - rendering lottie animations
* [copy-text-to-clipboard](https://www.npmjs.com/package/copy-text-to-clipboard) - copying codes to the clipboard

🏆 TeleOTP won first place in the [Telegram Mini App Contest](https://contest.com/mini-apps).

Designed by [@lunnaholy](https://github.com/lunnaholy),
implemented by [@LowderPlay](https://github.com/LowderPlay) & [@lenchq](https://github.com/lenchq) with ❤️
