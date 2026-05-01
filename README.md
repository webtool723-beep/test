# Green Cucina — Website Package

## Overview
Professional multi-page website for **Green Cucina Café & Kitchen**, Kota, Rajasthan.

---

## File Structure

```
green-cucina/
├── index.html          ← Main homepage
├── menu.html           ← Full menu page
├── css/
│   ├── style.css       ← Main stylesheet
│   └── menu.css        ← Menu page styles
├── js/
│   └── main.js         ← All interactivity
└── README.md           ← This file
```

---

## Pages & Features

### index.html (Homepage)
- **Navigation** — Fixed, transparent-to-dark on scroll; mobile hamburger menu
- **Hero Section** — Full-screen with animated background & floating leaves
- **Marquee Strip** — Scrolling highlights strip
- **About Section** — Story, stats & visual panel
- **Menu Preview** — Tabbed, filterable menu cards (All / Breakfast / Mains / Drinks / Desserts)
- **Experience Strip** — 4 value pillars on dark green background
- **Gallery** — Masonry-style image gallery grid
- **Reservation Form** — Full booking form with validation & confirmation toast
- **Map + Contact** — Embedded Google Maps + contact info panel
- **Footer** — Brand info, navigation, hours

### menu.html (Full Menu)
- Full menu with all categories
- Back-navigation to homepage

---

## Customisation Guide

### 1. Update Contact Details
In `index.html` and `menu.html`, search for:
- `+91 98765 43210` → Replace with real phone number
- `hello@greencucina.in` → Replace with real email
- `Near Dadabari, Kota, Rajasthan — 324009` → Update exact address

### 2. Add Real Photos
Replace the `.gallery-placeholder` divs in the gallery section with actual `<img>` tags:
```html
<img src="images/your-photo.jpg" alt="Description" />
```

Store photos in the `/images/` folder.

### 3. Update Prices
Menu prices are in `index.html` and `menu.html`. Each price has the class `.price`.

### 4. Google Maps Embed
The map in `index.html` already points to Green Cucina's coordinates (25.1445, 75.8582). 
To update, replace the `src` in the `<iframe>` tag.

### 5. Reservation Form Integration
Currently the form shows a confirmation toast. To make it functional:
- **WhatsApp**: Add `wa.me/91XXXXXXXXXX` link
- **Email**: Use FormSubmit.co or Formspree (free services)
- **Backend**: Connect to your own server/API

---

## Fonts Used
- **Cormorant Garamond** — Headings (Google Fonts)
- **DM Sans** — Body text (Google Fonts)

Loaded from Google Fonts CDN. Works with internet connection.

---

## Browser Support
- Chrome, Firefox, Safari, Edge (modern versions)
- Mobile responsive: 480px, 768px, 1024px breakpoints

---

## Deployment Options
1. **Hostinger / GoDaddy**: Upload all files via File Manager
2. **Netlify**: Drag & drop the folder at netlify.com/drop
3. **GitHub Pages**: Push to a repo, enable Pages in Settings

---

*Built for Green Cucina — Kota, Rajasthan. 2025.*
