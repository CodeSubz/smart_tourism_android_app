<a name="readme-top"></a>

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <img src="images/logo_name.jpg" alt="Logo" width="512" height="512">

  <p align="center">
    Smart Tourism app for Android Devices
  </p>
</div>

# Getting Started

<div align="center">
  <img src="images/reinherit_screen_athens.png" alt="example screenshot for Athens app" width="23%">
  &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;
  <img src="images/reinherit_screen_florence.jpg" alt="example screenshot for Florence app" width="70%"> 
</div>

The repository consists of two parts:

* Python (Docker supported)
* Android project

The Python part generates SQLite files from an image dataset and guides folder.  
The Android app uses these databases. Always recompile after Python-side changes.

---

## Monument Guides Creation

- Use the `Python/guides/CITYNAME` folder.
- Duplicate `Template Monument` for each real monument. Name must match dataset folder.
- Use [Markdown format](https://www.markdownguide.org/basic-syntax/) for content.

### Markdown Header Format:

<!-- METADATA <Latitude> <Longitude> <Category1>, <Category2>, <Category3> <Attribute1>, <Attribute2>, <Attribute3>, <Attribute4>, <Attribute5> <Subtitle> <Guide Link> -->


### Languages

- Add language folders (e.g., French, German).
- Keep folder and guide file names consistent.
- Each monument folder must have a guide in every supported language.

### Categories

- In `Python/categories/CITYNAME`, add **one image per category** used in guides.
- Image filenames = category names.

---
## Database creation

Complete the previous step before creating the database. The guides have to be completed and if you
make any modification in any guide you have to create again the database.

The repository contains the folder `Python/imageDatasets/CITYNAME` which must contains one folder per
monument and each of which contains the images, as in the following example:

```
imageDatasets/CITYNAME
├───Battistero SanGiovanni
│       img1.jpg
│       img2.jpg
│       img3.jpg
│
├───Campanile Giotto
│       img1.jpg
│       img2.jpg
│       img3.jpg
│
├───Cattedrale Duomo
│       img1.jpg
│       img2.jpg
│       img3.jpg
│
└───Palazzo Vecchio
       img1.jpg
       img2.jpg
       img3.jpg

```

Go to the folder containing the Dockerfile (i.e. the project folder) and run the following commands in a terminal:

```sh
1. docker build -t tfimage .
2. docker run -it tfimage CITYNAME
3. docker container ls -all
4. docker cp containerID:/app/models/src/main/assets/databases ./models/src/main/assets/
```
NOTE: You can find containerID in the list of the containers in the third instruction.

The following command runs the Docker to process tha visual features of the images and create the database.
The copy parameter copies also the guides and categories folders in the assets folder of the Android project.

```sh
1. ./run_docker.sh CITYNAME copy
```

otherwise run:

```sh
1. ./run_docker.sh CITYNAME
2. ./copy_guides.sh CITYNAME
```

to run first the Docker and then copy the guides and assets of the CITYNAME city (e.g. Florence, Nicosia, Athens).

If you want to update the guide without updating the visual features of the images, run:

```sh
1. ./copy_guides.sh CITYNAME
```

after updating the guides of interest, to reduce the time needed to create the database.

NOTE: 'CITYNAME' is the name of the city of the guide you are building and that you created in previous steps.
If the CITYNAME is not provided the guide for Florence will be built as default.


### APK

If you want to generate a new APK file, please refer to the following
guide: [How to Generate APK and Signed APK Files in Android Studio](https://code.tutsplus.com/tutorials/how-to-generate-apk-and-signed-apk-files-in-android-studio--cms-37927)

You will find the APK in `app/build/outputs/apk/support/debug` .

# App User Guide

Welcome to the app! This guide will help you understand the general functioning of the application
and how to use its features effectively. Please note that this guide is written for non-expert
users, so the instructions are simplified for better understanding.

## Installation and Setup

- Install the app on your Android device from the provided APK file. You can find the APK file in
  the directory: `app/build/outputs/apk/support/debug`.
- Once installed, open the app to begin the setup process.

## Loading Screen

- When you launch the app, a loading screen will appear. This screen allows the app to load the
  necessary data into memory. Please wait until the loading process is complete.
- If it's your first time accessing the application, you will be asked to select your preferred
  categories and attributes (optional).
- For the correct functioning of the application, please accept all the asked permissions.

## Main Screen

- After the loading process, you will be directed to the main screen of the app.
- On this screen, you will see a list of monuments, grouped by categories.
- The categories displayed are based on your preferred choices. You can modify your preferred
  categories later if needed.
- The monuments are listed in order of preference, considering their attributes and interaction
  history.

## Camera and Monument Recognition

- To use the camera for monument recognition, tap on the FAB (Floating Action Button) located at the
  bottom right corner of the main screen.
- Aim your device's camera at a monument and wait for the app to recognize it.
- If the app successfully recognizes a monument, a popup will appear with additional information
  about the monument.

## Monument Guides

- When you open a guide enjoy the experience!
- Moreover, the app will recommend three monuments to visit.
- If you tap on any of the recommended monuments, the app will open the corresponding guide for that
  monument. Open the integrated map to see how to reach it!

## Preferences

You have the option to modify your preferred categories and attributes in the settings. Simply
navigate to the preferences section and make the desired changes.
Feel free to explore the settings section and customize the app to your liking.

This guide provides an overview of the app's features and functionalities. Use the app to explore
and discover various monuments. If you have any further questions or need assistance, please refer
to the app's support documentation or contact me. Enjoy your journey!
<!-- CONTACT -->

