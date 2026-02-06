# Image Perception Study - Randomisation Interface

This repository contains the web-based randomisation interface for the research study:

**"User Perception of AI Generated Images"**  
*Authors: Ibrahim Murtaza, Khadija Imran, Xeerak Azhar, Momina Sajid*  
*Institution: Lahore University of Management Sciences (LUMS)*  
*Course: CS 6303 - Topics in Large Language Models*

## Overview

This is a custom web interface that implements a within-subjects experimental design for testing how content labels affect image perception. The system randomly generates unique image sequences for each participant, ensuring balanced exposure to all experimental conditions.

## Purpose

The interface was designed to:
1. Randomly select 2 AI-generated and 2 human-created images from pre-defined pools
2. Randomly assign labels ("AI-Generated" or "Human-Created") to create balanced experimental conditions
3. Display images sequentially with prominent labels
4. Log all assignments to Google Sheets for downstream analysis
5. Randomise presentation order to prevent order effects

## Experimental Design

Each participant sees exactly **4 images** representing all combinations of origin and label accuracy:

| Condition | Image Origin | Label Shown | Code |
|-----------|-------------|-------------|------|
| True Positive (TP) | AI | "AI-Generated" | ✓ Correct |
| True Negative (TN) | Real | "Human-Created" | ✓ Correct |
| False Positive (FP) | Real | "AI-Generated" | ✗ Mislabeled |
| False Negative (FN) | AI | "Human-Created" | ✗ Mislabeled |

This within-subjects design ensures each participant serves as their own control.

## Repository Structure

```
├── index.html                   # Main interface (randomization + display)
├── images/
│   ├── ai/                      # AI-generated images (n=10)
│   │   ├── apples.png
│   │   ├── earthquake.png
│   │   ├── feeding_the_poor.png
│   │   ├── football.png
│   │   ├── mountains.png
│   │   ├── stars.png
│   │   ├── trekker.png
│   │   ├── trump.png
│   │   ├── tsunami.png
│   │   └── wildfire.png
│   └── real/                    # Human-created images (n=10)
│       ├── cups.png
│       ├── flood.png
│       ├── football.png
│       ├── galaxy.png
│       ├── ladies_with_bags.png
│       ├── mountains.png
│       ├── mountain_clouds.png
│       ├── refugees.png
│       ├── trekker.png
│       └── wildfire.png
└── README.md                    # This file
```

## How It Works

### 1. Participant Entry
- Participant enters email address on start screen
- Email serves as unique identifier for linking web viewing data with survey responses

### 2. Randomization Algorithm

The JavaScript algorithm executes these steps:

```javascript
// Step 1: Shuffle both image pools
const shuffledReal = shuffle([...realImagesPool]);
const shuffledAI = shuffle([...aiImagesPool]);

// Step 2: Select top 2 from each
const selectedReal = shuffledReal.slice(0, 2);
const selectedAI = shuffledAI.slice(0, 2);

// Step 3: Assign conditions
// - 1 Real → "Human-Created" label (TN)
// - 1 Real → "AI-Generated" label (FP)
// - 1 AI → "AI-Generated" label (TP)
// - 1 AI → "Human-Created" label (FN)

// Step 4: Shuffle final presentation order
trials = shuffle(trials);
```

This guarantees:
- Each participant sees unique images (low probability of duplicates across 117 participants)
- Balanced exposure to all 4 conditions
- Randomised presentation order

### 3. Image Display
- Each image displays with label prominently below it
- Participant observes image naturally (no time limit)
- Clicks "Next" to proceed to the next image

### 4. Data Logging

After viewing all 4 images, the interface sends data to Google Apps Script web app endpoint:

**Data logged**:
```json
{
  "userEmail": "participant@email.com",
  "timestamp": "2025-11-15T14:32:01.123Z",
  "trials": [
    {
      "image": "images/ai/stars.png",
      "label": "AI-Generated",
      "condition": "TP (True Positive)",
      "type": "AI"
    },
    // ... 3 more trials
  ]
}
```

This data is matched with Qualtrics survey responses via email for analysis.

## Setup Instructions

### 1. Configure Google Apps Script Backend

The interface requires a Google Apps Script web app to log data.

**Script Code** (create in Google Apps Script):

```javascript
function doPost(e) {
  const sheet = SpreadsheetApp.openById('YOUR_SHEET_ID').getSheetByName('Sheet1');
  const data = JSON.parse(e.postData.contents);
  
  // Log each trial as a row
  data.trials.forEach((trial, index) => {
    sheet.appendRow([
      data.userEmail,
      data.timestamp,
      index + 1,
      trial.image,
      trial.label,
      trial.condition,
      trial.type
    ]);
  });
  
  return ContentService.createTextOutput('Success');
}
```

**Deploy as**:
- Web app
- Execute as: Me
- Who has access: Anyone

**Copy deployment URL** and paste into `index.html`:

```javascript
const SCRIPT_URL = "YOUR_GOOGLE_APPS_SCRIPT_URL_HERE";
```

### 2. Add Your Images

Place images in:
- `images/ai/` - AI-generated images
- `images/real/` - Human-created images

Update the image pools in `index.html`:

```javascript
const realImagesPool = [
  "images/real/cups.png",
  "images/real/flood.png",
  // ... add all real images
];

const aiImagesPool = [
  "images/ai/apples.png",
  "images/ai/earthquake.png",
  // ... add all AI images
];
```

## Data Export

**Google Sheets columns**:
```
| email | timestamp | trial_num | image_path | label_shown | condition | type |
```

This CSV can be merged with Qualtrics survey data for analysis.

## Features

### Design Considerations

✅ **Fully randomised**: No two participants likely see same sequence  
✅ **Balanced conditions**: Each participant acts as own control  
✅ **No memory bias**: Images shown without context or history  
✅ **Order effects controlled**: Random presentation sequence  
✅ **Logging**: All assignments tracked for reproducibility  
✅ **Mobile-friendly**: Responsive design works on tablets/phones  

### Technical Details

- **Frontend**: Pure HTML/CSS/JavaScript (no dependencies)
- **Backend**: Google Apps Script (serverless)
- **Storage**: Google Sheets (free, real-time)
- **Compatibility**: All modern browsers (Chrome, Firefox, Safari, Edge)

## Image Provenance

### Human-Created Images
- Sourced from publicly available repositories via Google Images
- Selected for content diversity (disasters, landscapes, people, objects)

### AI-Generated Images
**Pipeline**:
1. Human photograph → GPT-5 (descriptive JSON)
2. JSON description → Google Gemini API (image generation)

This ensures AI images match real photos in content and complexity.

## Limitations

- Requires internet connection for Google Sheets logging
- No offline mode (data must be logged during viewing)
- Email-based linking assumes participants use consistent email
- Randomisation is pseudo-random (JavaScript Math.random)

## Citation

If you use or adapt this interface, please cite:

```bibtex
@unpublished{Murtaza2025ImagePerception,
  title={User Perception of AI Generated Images},
  author={Murtaza, Ibrahim and Imran, Khadija and Azhar, Xeerak and Sadjid, Momina},
  year={2025},
  institution={Lahore University of Management Sciences},
  note={CS 6303 Final Project - Randomisation Interface}
}
```

## Related Repository

- **Data Analysis Code**: https://github.com/ibrahim-murtaza/image-perception-study-data-analysis

## License

This code is released for academic research purposes. Please contact authors for commercial use.
