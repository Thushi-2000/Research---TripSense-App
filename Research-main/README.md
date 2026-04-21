# TripSense Mobile App
A sentiment-aware hotel recommendation feature integrated into the **TripSense** mobile application for Sri Lankan tourism. This component recommends accommodations to travellers based on **real guest review sentiment scores**, hotel ratings, price, amenities, and CNN-powered category classification.

---

## Overview

This component is part of a larger group research project on *"Sarcasm-Aware Multimodal Location-Based Sentiment Analysis for Tourism"*. While the other components focus on analysing tourist location sentiment, **this component uses those sentiment outputs to recommend the best nearby hotels** to travellers visiting those locations.

The recommendation engine runs on a **FastAPI backend** (port 8000) with a **MongoDB** database, and the frontend is built in **React Native with Expo**.

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│              Mobile App (React Native Frontend)         │
│                                                         │
│  HotelFiltersScreen ──► HotelRecommendationsScreen      │
│         │                        │                      │
│         ▼                        ▼                      │
│   HotelDetailScreen ──► Booking Flow (3 steps)          │
│         │                                               │
│         └──────────── hotelApiService.js ───────────────┤
└─────────────────────────────────────────────────────────┘
                              │ HTTP/REST (port 8000)
┌─────────────────────────────▼───────────────────────────┐
│           FastAPI Backend (server.py — port 8000)        │
│                                                         │
│  POST /api/recommendations   GET /api/hotels            │
│  GET  /api/hotels/<id>       GET /api/hotel-category    │
│  GET  /api/hotels/search     GET /api/hotels/stats      │
│  GET  /api/hotel-images/<id>/list                       │
│  GET  /api/ai-insights/<id>  POST /api/feedback         │
│                                                         │
│  ┌──────────────────────┐   ┌──────────────────────┐   │
│  │   MongoDB Database   │   │   CNN (VGG) Model    │   │
│  │   (hotel data,       │   │   (image-based       │   │
│  │   reviews, users)    │   │   category ranking)  │   │
│  └──────────────────────┘   └──────────────────────┘   │
└─────────────────────────────────────────────────────────┘
```

---

## Features

### Hotel Discovery
- **Home Screen** (`HotelFiltersScreen`): Dark navy header with live hotel search, autocomplete suggestions, and categorised horizontal carousels:
  - *Recommended For You* — personalised using user ID & preferences
  - *Top Beach Hotels* — ranked by CNN image classification + sentiment
  - *Luxury Stays* — ranked by CNN image classification + rating
  - *Budget Friendly* — ranked by value score, without luxury duplicates
- **Real-time search** with 300ms debounce and instant autocomplete dropdown

### Sentiment-Aware Recommendations
- Each hotel card shows a **"% Positive"** badge derived from real NLP sentiment analysis of guest reviews
- Recommendations are personalised using a `user_id` (real user when logged in; stable anonymous ID otherwise)
- The backend `/api/recommendations` accepts travel `preferences` for cold-start personalisation

### Hotel Detail (`HotelDetailScreen`)
Four-tab detail screen for every hotel:

| Tab | Contents |
|---|---|
| **Overview** | Hotel description, guest sentiment breakdown (Positive / Neutral / Negative bar chart), contact info (maps, website, phone) |
| **Amenities** | Emoji-icon grid for all amenities (Pool, Spa, WiFi, Beach Access, etc.) |
| **Reviews** | Up to 10 guest reviews with star rating and review date |
| **AI Insights** | Gemini-powered summary of hotel strengths and weaknesses (cached 7 days on backend) |

Additional features on the detail screen:
- Scrollable image gallery with thumbnail strip and counter
- Liked/favourited state persisted to `AsyncStorage` and synced to backend via `/api/feedback`
- Animated toast notifications (e.g., "Added to favourites")
- **Book Now** button → three-step booking flow

### Location-Based Recommendations (`HotelRecommendationsScreen`)
- Triggered from the sentiment map when a user taps a location
- Includes a **landmark → city resolver** that maps attraction names to the nearest city where hotels are stored:
  - e.g., *"Temple of the Sacred Tooth Relic"* → `Kandy`
  - e.g., *"Nine Arch Bridge"* → `Ella`
  - 17 Sri Lankan landmark groups mapped
- Supports both `location`-based and `category`-based hotel lists
- Shows a **"% Match"** badge (computed from `avg_sentiment_score`)

### Booking Flow (3 Steps)
1. **Booking Details** (`BookingDetailsScreen`): Select check-in/check-out dates (date picker), number of guests, number of rooms — live total price calculation
2. **Guest Information** (`BookingGuestInfoScreen`): Enter guest name, email, phone number
3. **Confirmation** (`BookingConfirmationScreen`): Booking summary with a redirect to the hotel's website or Booking.com

---

## Installation

### Backend Setup (FastAPI — port 8000)

The hotel backend is a **FastAPI** server using MongoDB. It is separate from the Flask sentiment backend.

1. Navigate to the backend directory and set up a virtual environment:
```bash
cd backend
python -m venv venv
venv\Scripts\activate        # Windows
```

2. Install dependencies:
```bash
pip install -r requirements.txt
```

3. Set up environment variables (copy `.env.example` to `.env` and fill in):
```bash
copy .env.example .env
```
Key variables needed:
```
MONGODB_URL=mongodb://localhost:27017
MONGODB_DB=hotel_recommendations
GOOGLE_MAPS_API_KEY=your_key_here
GEMINI_API_KEY=your_key_here
```

4. Start the FastAPI backend:
```bash
python server.py
# or: uvicorn server:app --host 0.0.0.0 --port 8000 --reload
```

The server starts on `http://localhost:8000`. Verify at `http://localhost:8000/docs`.

---

### Frontend Setup (React Native)

1. Install Node.js dependencies from the project root:
```bash
npm install
```

2. Configure the hotel backend IP in `src/services/hotelApiService.js`:
```javascript
const COMPUTER_IP = '192.168.x.x'; // Your PC's LAN IP (run ipconfig)
```

> **Tip**: Run `CHECK_IP.bat` to quickly find your LAN IP address.

The service automatically selects the correct URL per platform:
- **Android emulator**: `http://<COMPUTER_IP>:8000/api`
- **iOS simulator**: `http://<COMPUTER_IP>:8000/api`
- **Web**: `http://localhost:8000/api`

You can also set the IP via an environment variable:
```bash
EXPO_PUBLIC_COMPUTER_IP=192.168.1.100 npx expo start
```

3. Start the app:
```bash
npm start         # Expo dev server
npm run android   # Android
npm run ios       # iOS (macOS only)
npm run web       # Web browser (for testing)
```

---

## Project Structure (Component Files)

```
Research-main/
│
├── src/
│   ├── screens/
│   │   ├── HotelFiltersScreen.js            # Hotel home screen (search + categories)
│   │   ├── HotelRecommendationsScreen.js    # Location/category-based hotel list
│   │   ├── HotelRecommendationsListScreen.js # Full hotel list view
│   │   ├── HotelsScreen.js                  # Simple hotel browse screen
│   │   ├── HotelDetailScreen.js             # Hotel detail (4-tab: Overview/Amenities/Reviews/AI)
│   │   ├── BookingDetailsScreen.js          # Step 1: Dates, guests, rooms
│   │   ├── BookingGuestInfoScreen.js        # Step 2: Guest name, email, phone
│   │   └── BookingConfirmationScreen.js     # Step 3: Booking summary & redirect
│   │
│   ├── components/
│   │   └── RecommendationCard.js            # Hotel card used in recommendation lists
│   │
│   └── services/
│       └── hotelApiService.js               # All API calls to the FastAPI hotel backend
│
├── backend/                                 # Flask sentiment backend (port 5000)
│   └── app.py                               # Contains stub hotel endpoints (returns [] when
│                                            # FastAPI hotel backend is offline)
│
├── CHECK_IP.bat                             # Show your PC's LAN IP
├── SETUP_FIREWALL.bat                       # Allow port 8000 through Windows Firewall
└── START_TUNNEL.bat                         # ngrok tunnel for remote device testing
```

---

## API Reference

All endpoints are served by the **FastAPI backend on port 8000** (`/api` prefix).

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/recommendations` | Personalised hotel recommendations by location, preferences, min rating |
| `GET` | `/api/hotels` | List all hotels (optionally filter by `location`, `min_rating`, `limit`) |
| `GET` | `/api/hotels/<hotel_id>` | Full hotel details including reviews and amenities |
| `GET` | `/api/hotels/search?q=` | Autocomplete search by hotel name or city |
| `GET` | `/api/hotels/stats` | Aggregate stats: total hotels, average rating |
| `GET` | `/api/hotel-category?type=beach&limit=12` | Hotels ranked by CNN (VGG) image model for `beach`, `luxury`, or `budget` |
| `GET` | `/api/hotel-images/<hotel_id>/list` | All image URLs for a hotel |
| `GET` | `/api/ai-insights/<hotel_id>` | Gemini AI summary of hotel strengths/weaknesses (7-day cache) |
| `POST` | `/api/feedback` | Submit a favourite/rating signal (`rating: 5.0 = like, 1.0 = unlike`) |

### Recommendation Request Body
```json
{
  "user_id": "123",
  "location": "Kandy",
  "limit": 30,
  "amenities": ["Swimming Pool", "Free WiFi"],
  "min_rating": 3.5,
  "preferences": ["beach", "luxury"]
}
```

### Hotel Response Fields
```json
{
  "hotel_id": 42,
  "name": "Cinnamon Grand Kandy",
  "location": "Kandy",
  "image_url": "https://...",
  "rating": 4.7,
  "total_reviews": 312,
  "avg_sentiment_score": 0.68,
  "positive_pct": 84,
  "positive_reviews": 262,
  "neutral_reviews": 31,
  "negative_reviews": 19,
  "price_info": { "min": 95, "max": 220, "tier": "luxury" },
  "amenities": ["Swimming Pool", "Spa & Wellness", "Restaurant"],
  "description": "...",
  "website": "https://...",
  "phone": "+94 81 234 5678",
  "google_maps_url": "https://maps.google.com/..."
}
```

---

## Key Technical Details

### Sentiment-Based Ranking
- `avg_sentiment_score` (–1.0 to 1.0): Average NLP sentiment across all hotel reviews
- `positive_pct`: Percentage of reviews labelled *positive* — shown as the **"% Positive"** badge on hotel cards
- `positive_reviews`, `neutral_reviews`, `negative_reviews`: Raw counts used for the sentiment breakdown bar chart in the Overview tab

### CNN Image Classification
- Hotel categories (beach / luxury / budget) are determined server-side using a **VGG CNN model** trained on hotel images
- Frontend calls `/api/hotel-category?type=beach` — no client-side keyword guessing
- Luxury and budget lists are deduplicated client-side so a hotel appears in only one category

### User Personalisation
- Logged-in users: `global.userId` (integer from Flask SQLite auth) is cast to string and sent as `user_id`
- Guest users: A stable anonymous ID (`anon-<timestamp>-<random>`) is generated per session and reused for consistent cold-start results
- Travel preferences stored in `global.userPreferences` are passed to the backend for cold-start personalisation

### Liked Hotels (Favourites)
- Liked hotel IDs are stored in `AsyncStorage` (key: `liked_hotels`) as a JSON array
- Optimistic UI update: the heart icon flips immediately, then the backend `/api/feedback` call is made
- IDs are stored as both `Number` and `String` to handle type mismatches across API responses

### Landmark → City Resolver
The `resolveNearbyCity()` function in `HotelRecommendationsScreen.js` maps 17 Sri Lankan landmark groups to searchable city names:

```javascript
{ keywords: ['tooth relic', 'kandy lake', 'peradeniya'], city: 'Kandy' }
{ keywords: ['sigiriya', 'lion rock', 'dambulla'],       city: 'Sigiriya' }
{ keywords: ['nine arch', 'ella', 'ravana falls'],       city: 'Ella' }
// ... 14 more groups
```

This ensures that when a user taps a tourist attraction on the sentiment map, the hotel search finds results for the nearest city rather than returning no results.

### AI Insights (Gemini)
- The backend calls the **Google Gemini API** to generate a structured paragraph summarising a hotel's strengths, weaknesses, and value proposition based on its reviews
- Results are cached for **7 days** on the backend to avoid re-calling the Gemini API
- Displayed in the "AI Insights" tab of `HotelDetailScreen`

---

## Technical Stack

| Layer | Technology |
|---|---|
| Mobile Framework | React Native 0.81 with Expo 54 |
| Navigation | React Navigation (Stack + Bottom Tabs) |
| UI / Icons | Expo Vector Icons (Ionicons) · Expo Linear Gradient |
| Persistent Storage | `@react-native-async-storage/async-storage` |
| Date Picker | `@react-native-community/datetimepicker` |
| Backend Framework | FastAPI (Python) |
| Database | MongoDB |
| AI Insights | Google Gemini API |
| Image Classification | CNN / VGG model (hotel category ranking) |
| Maps / Places | Google Maps API (hotel location & contact data) |

---

## Troubleshooting

### "Cannot reach hotel backend"
1. Run `CHECK_IP.bat` to find your PC's IP address
2. Update `COMPUTER_IP` in `src/services/hotelApiService.js`
3. Run `SETUP_FIREWALL.bat` to allow port 8000 through Windows Firewall
4. Ensure the FastAPI server is running: visit `http://localhost:8000/docs`

> **Note**: The Flask backbone (port 5000) contains **stub endpoints** for `/api/hotels` and `/api/recommendations` that return empty arrays (`[]`). This means the app will not crash if the hotel backend is offline — it will simply show an empty hotels screen. For real hotel data, the FastAPI server on port 8000 must be running.

### No Hotels Showing
1. Confirm the FastAPI backend is running on port 8000
2. Ensure MongoDB is running and the database is populated with hotel data
3. Check the Expo logs for `[Hotel API]` messages showing the URL being called

### AI Insights Tab Shows Error
1. Verify `GEMINI_API_KEY` is set correctly in the backend `.env` file
2. The Gemini API requires an active internet connection on the backend server
3. The cache is per `hotel_id` — if a hotel has never been requested, the first call may take a few seconds

### Recommendations Not Personalised
- On first launch, an anonymous session ID is used — this is expected
- After sign-in, ensure `global.userId` is set in the auth flow (`authService.js`)
- Travel preferences in `global.userPreferences` must be set before calling the recommendations endpoint

---

## Research Integration

This component integrates with the broader research project as follows:

- **Sentiment scores from the Flask backend** (`TripSense`) are surfaced alongside hotel recommendations — when a user views a tourist location's sentiment detail, they are offered a "Find Hotels near here" button that triggers `HotelRecommendationsScreen` with the location name
- **Hotel review sentiment** (positive / neutral / negative breakdowns) is computed by the FastAPI backend using the same NLP methodology as the location sentiment pipeline
- **CNN image classification** ranks hotels by visual category (beach, luxury, budget), complementing the text-based sentiment analysis in the other research components
