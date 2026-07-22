## hw4a-student

> **Please read this handout from start to finish before proceeding to work on the assignment**

# CIS 5500: Database and Information Systems
## Homework 4a: WebDB

**Please read this handout from start to finish before proceeding to work on the assignment**

---

## Introduction

Homework 4 is divided into two sections (Homework 4a which is worth 80 points and Homework 4b which is worth 20 points), each of which is to be submitted separately to Gradescope by their due dates. This document discusses the first section, in which you will incrementally build an interactive Spotify-themed web application using React and Node.js backed by a PostgreSQL RDS database. The second section is a written assignment on B+ trees.

Homework 4a will help you understand the fundamentals of web interface design and implementation. In the first part (Part 1, worth 40 points) you will build an API on Node.js following the specification that we lay out for you (outlined below). This will help you understand how APIs are able to process and facilitate data interchange between various (client) applications and databases.

Once you familiarize yourself with the two 'lowest' tiers of the architecture (the API server and database), you will use your API to develop a frontend application using React.js that uses it. Specifically, in Part 2, worth 40 points, you will develop an interactive multi-page client using UI component libraries MUI (based on Google's Material Design system) and recharts (for data visualization).

Part 2 of Homework 4a primarily serves as a primer on front-end web development. By the end of the assignment, we hope you will have a great template that you can refer to, modify, and use for the project. It is developed to be similar to a 'follow-along' exercise on React and its component libraries. In fact, we provide you with most of the code and many examples, and the short tasks ask you to fill in or correct some of this implementation. In doing so, you will have the benefit of learning these (much needed) web development skills while avoiding a steeper learning curve.

---

## Application Structure

Unzip HW3_student.zip from the assignment page. You should have the following files (please read over the explanations for each file):

### /server

This folder holds the server application files, tests, and dependencies (as required by Node.js).

- **.gitignore**: A gitignore file for the Node application. Read more on .gitignore files [here](https://git-scm.com/docs/gitignore).
- **config.json**: Holds the RDS connection credentials/information and application configuration settings (like port and host). **You will need to edit this file with your database credentials.**
- **package.json**: maintains the project dependency tree; defines project properties, scripts, etc
- **package-lock.json**: saves the exact version of each package in the application dependency tree for installs and maintenance.
- **routes.js**: This is where the code for the API routes' handler functions go. We have already defined the necessary routes for you - follow the 'TODO:' comments and implement/modify them as specified). **You will need to edit this file and complete the labeled tasks.**
- **server.js**: The code for the routed HTTP application. You will see that it imports routes.js and maps each route function to an API route and type (like GET, POST, etc). For this HW, we will only use GET requests. It also 'listens' to a specific port on a host using the parameters in config.json.

### /server/__tests__

This folder contains the test files for the API:

- **results.json**: Stores (some) expected results for the tests in a json encoding.
- **tests.js**: Runs the test cases using the Jest JavaScript testing library.

### /client

- **.gitignore**: A gitignore file for the client application. Read more on .gitignore files [here](https://git-scm.com/docs/gitignore).
- **package.json**: maintains the project dependency tree; defines project properties, scripts, etc
- **package-lock.json**: saves the exact version of each package in the application dependency tree for installs and maintenance

### /client/public

This folder contains static files like index.html file and assets like robots.txt for specifying web page titles, crawlability, et cetera (more info [here](https://create-react-app.dev/docs/using-the-public-folder/)).

### /client/src

This folder contains the main source code for the React application. Specifically:

- **App.js**: This holds the root component of the React application and provides the theme of the application (using MUI).
- **config.json**: Holds server connection information (like port and host). Do not edit unless for some reason you change the default "localhost" and port "8080" in /server/config.json.
- **index.js**: This the main JavaScript entry point to the application and stores the main DOM render call in React. For this application, page routing via components and imports for stylesheets are also embedded in this file.
- **/helpers**: This folder contains a JavaScript file to format data:
  - **formatter.js**: Provides two functions to format times and dates
- **/components**: Similar to the /pages folder, but this folder contains files for React components corresponding to smaller, reusable components, especially those used by pages.
  - **LazyTable.js**: A lazy loading (only fetches the rows it needs) table leveraging MUI's basic table component with syntax very similar to MUI's DataGrid component. **You will need to edit this file and complete the labeled tasks.**
  - **NavBar.js**: Top navigation bar to navigate between different pages of the app.
  - **SongCard.js**: Modal containing information about a selected song. **You will need to edit this file and complete the labeled tasks.**
- **/pages**: This folder contains files for React components corresponding to the four pages in the application (see the sections below for more details). These are:
  - **AlbumsInfoPage.js**: A page to show information about a specific album. **You will need to edit this file and complete the labeled tasks.**
  - **AlbumsPage.js**: A page that lists all albums and contains links to album specific pages (defined in AlbumsInfoPage.js). **You will need to edit this file and complete the labeled tasks.**
  - **HomePage.js**: The landing page for the app. Contains among other things a song of the day and tables with the top songs and albums, with links to relevant pages. **You will need to edit this file and complete the labeled tasks.**
  - **SongsPage.js**: A page to search and filter results for songs based on various parameters (name, duration, plays, danceability, energy, valance, explicit). **You will need to edit this file and complete the labeled tasks.**

---

## Dataset Overview

The full DDL statements for the database are copied below. Please take a look to understand the structure of the dataset. **Do not run these statements.**

```sql
CREATE TABLE Albums (
    album_id varchar(22) PRIMARY KEY,
    title varchar(32),
    release_date varchar(10),
    thumbnail_url varchar(64)
);

CREATE TABLE Songs (
    song_id varchar(22) PRIMARY KEY,
    album_id varchar(22),
    title varchar(128),
    number int,
    duration int,
    plays int,
    danceability float,
    energy float,
    valence float,
    tempo int,
    key_mode varchar(8),
    explicit int,
    FOREIGN KEY (album_id) REFERENCES Albums (album_id)
);
```

Most columns in the dataset are fairly self explanatory. A couple notable attributes of Songs that may not be obvious are:

- **Songs.number**: track number on the album
- **Songs.duration**: length of song in seconds
- **Songs.danceability**: float from 0 to 1 representing how danceable the song is as according to Spotify
- **Songs.energy**: similar, but representing how energetic the song is
- **Songs.valence**: similar, but representing how happy the song is
- **Songs.explicit**: 1 if the song is explicit, 0 otherwise

---

---
> Source: [Albinator3000/hw4a_student](https://github.com/Albinator3000/hw4a_student) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
