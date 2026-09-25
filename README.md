# Intelligent Lost and Found Recovery Platform

An intelligent web-based **Lost and Found Recovery Platform** built with Django that helps users report lost and found items and automatically identify potentially matching items using **text and image similarity**.

The platform combines traditional web application functionality with computer vision and similarity-based matching to make the recovery process more organized and efficient.

---

## Overview

Traditional lost-and-found systems often rely on manual reporting and searching. This project provides a centralized platform where authenticated users can:

* Report items they have lost
* Report items they have found
* Upload item images
* Search for potential matches
* Compare lost and found items using text and image characteristics
* Receive email notifications when a potentially matching found item is identified
* Track completed matches through the system

The matching process considers multiple characteristics of an item rather than relying only on its name.

---

## Key Features

### User Authentication

* Django authentication system
* User login and logout
* Separate handling for regular users and staff users
* Staff users are redirected to the Django administration panel
* Authenticated users can access the main platform

### Report Lost Items

Users can submit:

* Item name
* Location
* Category
* Description
* Date
* Optional image

The uploaded image is stored using Cloudinary.

### Report Found Items

Users can report found belongings using:

* Item name
* Location
* Category
* Description
* Date
* Image

When a found item is submitted, the system checks existing lost-item reports for potential matches.

### Intelligent Item Matching

The system compares lost and found items using multiple similarity techniques:

1. **Text similarity**
2. **Image color similarity**
3. **Perceptual image hashing**
4. **Location matching**
5. **Category matching**
6. **Date matching**

These signals are combined to produce an overall similarity score.

### Email Notifications

When a newly reported found item is sufficiently similar to an existing lost item and satisfies the matching conditions, the system sends an email notification to the user who reported the lost item.

The notification contains details about the potentially matching found item.

### Match Confirmation

When a lost and found item are confirmed as a match, the system creates a `MatchedItem` record containing:

* Lost item
* Found item
* Collection status
* Match date
* Person who collected the item

---

# Intelligent Matching System

The core of the project is the `comparedetails()` matching function.

The system calculates three major forms of similarity.

## 1. Text Similarity

The descriptions of the lost and found items are compared using **Levenshtein distance**.

The distance is converted into a percentage-based similarity score.

```text
Text Similarity
       │
       ▼
Levenshtein Distance
       │
       ▼
Similarity Percentage
```

---

## 2. Color Histogram Similarity

The system processes both item images using OpenCV.

Each image is:

* Downloaded from its stored URL
* Converted to RGB
* Resized to `256 × 256`
* Separated into red, green and blue channels
* Converted into normalized color histograms

The histograms are then compared using OpenCV's histogram correlation method.

This produces a measure of similarity between the color distributions of the two images.

---

## 3. Perceptual Hash Similarity

The system also uses **perceptual hashing (pHash)** to compare the visual structure of the images.

The images are converted to grayscale and resized before generating their perceptual hashes.

The Hamming distance between the hashes is then converted into a similarity percentage.

This allows the system to compare images based on their overall visual structure rather than requiring the images to be identical.

---

## Combined Similarity Score

The three similarity components are combined using the following weighting:

```text
Overall Similarity =
    30% × Text Similarity
  + 40% × Image Color Similarity
  + 30% × Shape/Image Hash Similarity
```

This means image color similarity contributes the largest portion of the final score.

The implementation calculates this combined score in `reports/views.py`.

---

## Match Conditions

Similarity alone is not sufficient.

The application also compares:

* Location
* Category
* Date

For a lost item, potential matches are considered when the similarity score reaches the configured threshold and at least one of the location, category, or date conditions matches.

The matching thresholds differ between the initial lost-item matching page and the automatic notification flow.

---

# Application Flow

```text
                  ┌──────────────────────┐
                  │      User Login      │
                  └──────────┬───────────┘
                             │
                             ▼
                  ┌──────────────────────┐
                  │    Platform Home     │
                  └──────────┬───────────┘
                             │
                  ┌──────────┴───────────┐
                  │                      │
                  ▼                      ▼
          ┌───────────────┐      ┌───────────────┐
          │ Report Lost   │      │ Report Found  │
          │    Item       │      │     Item      │
          └───────┬───────┘      └───────┬───────┘
                  │                      │
                  ▼                      ▼
          ┌───────────────┐      ┌────────────────┐
          │ Store Item    │      │ Store Found    │
          │ Details       │      │ Item + Image   │
          └───────┬───────┘      └───────┬────────┘
                  │                      │
                  │                      ▼
                  │              ┌────────────────┐
                  │              │ Find Lost Items │
                  │              │ Same Category  │
                  │              └───────┬────────┘
                  │                      │
                  └──────────────┬───────┘
                                 ▼
                       ┌─────────────────────┐
                       │ Similarity Analysis │
                       │                     │
                       │ Text                │
                       │ Color Histogram     │
                       │ Perceptual Hash     │
                       └──────────┬──────────┘
                                  │
                                  ▼
                       ┌─────────────────────┐
                       │ Contextual Checks   │
                       │                     │
                       │ Location            │
                       │ Category            │
                       │ Date                │
                       └──────────┬──────────┘
                                  │
                                  ▼
                       ┌─────────────────────┐
                       │ Potential Match     │
                       └──────────┬──────────┘
                                  │
                                  ▼
                       ┌─────────────────────┐
                       │ Email Notification  │
                       └─────────────────────┘
```

---

# Technology Stack

| Technology               | Purpose                                   |
| ------------------------ | ----------------------------------------- |
| **Python**               | Core programming language                 |
| **Django 5.2**           | Web framework                             |
| **MySQL**                | Relational database                       |
| **PyMySQL**              | MySQL database connectivity               |
| **OpenCV**               | Image processing and histogram comparison |
| **Pillow**               | Image loading and processing              |
| **ImageHash**            | Perceptual image hashing                  |
| **python-Levenshtein**   | Text similarity calculation               |
| **NumPy**                | Numerical image processing                |
| **Cloudinary**           | Image storage                             |
| **WhiteNoise**           | Static file serving                       |
| **HTML/CSS**             | User interface                            |
| **Django Email Backend** | Email notifications                       |

The Django configuration registers the `accounts` and `reports` applications and integrates Cloudinary storage, WhiteNoise, MySQL, and Django's authentication system.

---

# Project Structure

```text
Intelligent-Lost-and-Found-Recovery-Platform/
│
├── LostandFound/
│   │
│   ├── LostandFound/
│   │   ├── settings.py
│   │   ├── urls.py
│   │   ├── asgi.py
│   │   └── wsgi.py
│   │
│   ├── accounts/
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── forms.py
│   │   ├── models.py
│   │   ├── admin.py
│   │   └── templates/
│   │
│   ├── reports/
│   │   ├── models.py
│   │   ├── views.py
│   │   ├── urls.py
│   │   ├── forms.py
│   │   ├── admin.py
│   │   ├── migrations/
│   │   ├── templates/
│   │   └── static/
│   │
│   └── manage.py
│
├── Procfile
└── .gitignore
```

The main Django URL configuration routes `/accounts/` to authentication and `/reports/` to the lost-and-found functionality.

---

# Database Models

The application contains three primary domain models.

### Lost

Stores information about an item reported as lost.

```text
Lost
├── user
├── item_name
├── location
├── category
├── item_desc
├── image_url
└── date
```

### Found

Stores information about an item reported as found.

```text
Found
├── user
├── item_name
├── location
├── category
├── item_desc
├── image_url
└── date
```

### MatchedItem

Records a confirmed relationship between a lost and found item.

```text
MatchedItem
├── lost_item
├── found_item
├── is_collected
├── date
└── collected_by
```

These models are implemented in the `reports` Django application.

---

# Main Routes

| Route                                     | Purpose                 |
| ----------------------------------------- | ----------------------- |
| `/accounts/login/`                        | User login              |
| `/accounts/logout/`                       | User logout             |
| `/reports/home/`                          | Main application home   |
| `/reports/reports/lost/`                  | Report a lost item      |
| `/reports/reports/found/`                 | Report a found item     |
| `/reports/check/<id>/`                    | Check potential matches |
| `/reports/success2/<lost_id>/<found_id>/` | Record a matched item   |

These routes are defined in the `accounts` and `reports` URL configurations.

---

# Installation

## Prerequisites

Make sure you have:

* Python 3.10+
* pip
* MySQL
* Git

## 1. Clone the Repository

```bash
git clone https://github.com/Sudheesh-jojo/Intelligent-Lost-and-Found-Recovery-Platform.git
cd Intelligent-Lost-and-Found-Recovery-Platform
```

## 2. Navigate to the Django Project

```bash
cd LostandFound
```

## 3. Create a Virtual Environment

```bash
python -m venv myenv
```

## 4. Activate the Environment

### Windows PowerShell

```powershell
myenv\Scripts\Activate.ps1
```

### Windows CMD

```cmd
myenv\Scripts\activate
```

## 5. Install Dependencies

```bash
pip install -r requirements.txt
```

## 6. Configure the Database

The application is configured to use **MySQL**.

Create the required database and configure the database connection before starting the application.

## 7. Configure External Services

The project uses:

* Cloudinary for image storage
* Gmail SMTP for email notifications

These credentials should be provided through environment variables rather than committed directly to the repository.

## 8. Run Migrations

```bash
python manage.py migrate
```

## 9. Create an Admin User

```bash
python manage.py createsuperuser
```

## 10. Start the Development Server

```bash
python manage.py runserver
```

Then open:

```text
http://127.0.0.1:8000/
```

---

# Security Note

Before deploying or sharing this project publicly:

* Move Django `SECRET_KEY` to an environment variable
* Move MySQL credentials to environment variables
* Move Cloudinary credentials to environment variables
* Move email credentials to environment variables
* Rotate any credentials that have previously been committed to Git

**Never commit API keys, passwords, database credentials, or email app passwords to GitHub.**

---

# Future Improvements

Potential improvements include:

* More advanced semantic text similarity
* Improved image feature extraction
* Better ranking of potential matches
* Configurable matching thresholds
* User-facing match confirmation workflow
* Improved notification management
* Better validation and error handling
* Environment-based configuration for deployment
* Automated tests for the matching pipeline
* REST API support for future frontend clients

---

# Author

**Sudheesh**

GitHub:
https://github.com/Sudheesh-jojo
