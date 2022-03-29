---
title: "Toki Pona Dictionary: A simple Toki Pona - English dictionary"
permalink: /toki-pona-dictionary-release
custom_date: "210405"
---

# Toki Pona Dictionary: A simple Toki Pona - English dictionary

**The dictionary is no longer online, I've decided to stop development of it. Feel free to fork, publish or modify it how you see fit, it is uncopyrighted**.

---

This is an online dictionary that I built for any person interested in the language Toki Pona. It used to be located at https://jprogr.github.io/TokiPonaDictionary/.

The dictionary includes all the words in a simple page, and a tool to search words and meanings at the same time and some notes of usage at the bottom. It strives to cover all the words as used by Toki Pona speakers without forgetting about the official dictionary. So it includes some unofficial words and meanings but they are always marked with an asterisk.

The code is open source, if you want to contribute or just take a look, you can find it at GitHub: [https://github.com/jProgr/TokiPonaDictionary](https://github.com/jProgr/TokiPonaDictionary). Also, feel free to point out any issues.

## Release history

The project does not follow semantic versioning due to its simplicity, so each new version is just the date of release written as “yymmdd”. The releases from newest to oldest:

### 210412

Changed:
- Added new word to the dictionary.

### 210217

Changed:
- The search algorithm now uses complete words instead of partial ones.
- Improved the page load: Now styles are loaded with the content and they are smaller.

Fixed:
- Fixed multiple definitions from the dictionary.
- Minor fixes to the main input on mobile devices.

### 200729

Changed:
- Changed behaviour on footer links.
- The project now uses different tooling in order to serve less bytes:
    - The page is now 87% smaller: From serving around 1536 KiB to 200 KiB.
    - Less requests: from 11 to 2 if fonts are found on the client machine.
    - Even if fonts need to be served they are optimized: From serving 1228 KiB to 20 KiB.
- Refactored search functionality: It is now faster and uses less memory.

### 200503

Changed:
- Updated dependencies.
- The dependencies are now loaded from the same source instead of some third party CDN.

### 200121

Fixed:
- Fixed multiple definitions from the dictionary.

### 191130

Added:
- A changelog will now be kept.

Changed:
- The "*" mark now has a better definition.
- Friendlier language.
- Updated some meanings and words of the dictionary according to The Official Toki Pona Dictionary (2014).
- Updated credits and description of the project.

### 191105

No changelog for this version.

### 191031

No changelog for this version.
