<div align="center">
<h3 align="center">U-There</h3>

  <p align="center">
    A PWA for creating groups and indicating online/offline status.
    <br />
     <a href="https://u-there.web.app">u-there.web.app</a>
  </p>
</div>

## Table of Contents

<details>
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#key-features">Key Features</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
  </ol>
</details>

## About The Project

U-There is a Progressive Web Application (PWA) built with Svelte and Firebase that allows users to create groups and indicate their online/offline status. This facilitates communication and coordination within teams. The application leverages Firestore (NoSQL) for data storage and real-time updates.

### Key Features

- **Group Creation:** Users can create groups with a name and optional image.
- **User Management:** Group admins can add or remove members from the group.
- **Admin Roles:** Group admins can assign or remove other admins.
- **Status Indication:** Users can indicate their online/offline status (Available, Busy, Offline, On Vacation).
- **Real-time Messaging:**  A simple chat interface is available within each group.
- **PWA Functionality:**  Installable and works offline thanks to service worker caching.
- **WhatsApp Integration:**  Users can initiate WhatsApp chats with other group members directly from the app.

The application utilizes the following technologies:

- **Frontend:** Svelte, Svelte Navigator
- **Backend:** Firebase (Authentication, Firestore)
- **PWA:** Service Worker
- **Build Tool:** Vite
- **Languages:** TypeScript, JavaScript, HTML, CSS

## Getting Started

### Prerequisites

-   Node.js and npm installed.
-   Firebase project set up with Firestore enabled.
-   Firebase configuration object (API key, etc.).

### Installation

Instructions for cloning the repo, installing packages, configuring environment variables, etc:

1.  Clone the repository:
    ```sh
    git clone https://github.com/breno-faria/u-there
    ```
2.  Navigate to the project directory:
    ```sh
    cd u-there
    ```
3.  Install dependencies:
    ```sh
    npm install
    ```
4.  Create a `.env` file in the root directory and add your Firebase configuration:
    ```
    VITE_apiKey=YOUR_API_KEY
    VITE_authDomain=YOUR_AUTH_DOMAIN
    VITE_projectId=YOUR_PROJECT_ID
    VITE_storageBucket=YOUR_STORAGE_BUCKET
    VITE_messagingSenderId=YOUR_MESSAGING_SENDER_ID
    VITE_appId=YOUR_APP_ID
    ```
    
5.  Run the development server:
    ```sh
    npm run dev
    ```
    This will start the application in development mode.
