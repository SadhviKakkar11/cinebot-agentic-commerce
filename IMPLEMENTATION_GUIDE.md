# Step-by-Step Implementation Guide

## Movie Ticket Booking Agent - Complete Build Instructions

This guide walks you through building an AI-powered movie ticket booking system using Claude and Python.

---

## 📋 Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Prerequisites & Setup](#prerequisites--setup)
3. [Step 1: Set Up Backend API](#step-1-set-up-backend-api)
4. [Step 2: Create Mock Data](#step-2-create-mock-data)
5. [Step 3: Build Claude Agent](#step-3-build-claude-agent)
6. [Step 4: Test the Integration](#step-4-test-the-integration)
7. [Step 5: Deploy & Scale](#step-5-deploy--scale)

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                   User Interface                             │
│        (Web UI, CLI, or Chat Interface)                     │
└──────────────────────┬──────────────────────────────────────┘
                       │
                       ▼
┌─────────────────────────────────────────────────────────────┐
│                  Claude AI Agent                             │
│        (Conversational AI with Tool Use)                    │
│                                                              │
│  - Understands user intent                                  │
│  - Decides which tools to call                              │
│  - Processes results                                        │
│  - Generates human-friendly responses                       │
└──────────────────────┬──────────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
    ┌────────┐   ┌─────────┐   ┌──────────┐
    │ Movies │   │ Theatres│   │ Bookings │
    │ Tools  │   │ Tools   │   │ Tools    │
    └────────┘   └─────────┘   └──────────┘
        │              │              │
        └──────────────┼──────────────┘
                       │
                       ▼
        ┌─────────────────────────────┐
        │   Backend REST API          │
        │   (Flask/FastAPI)           │
        │                             │
        │  - /api/movies              │
        │  - /api/theatres            │
        │  - /api/shows               │
        │  - /api/bookings            │
        └──────────────┬──────────────┘
                       │
                       ▼
        ┌─────────────────────────────┐
        │   Database                  │
        │   (In-memory or DB)         │
        │                             │
        │  - Movie catalog            │
        │  - Theatre data             │
        │  - Show schedules           │
        │  - Booking records          │
        └─────────────────────────────┘
```

---

## Prerequisites & Setup

### Requirements

- Python 3.8 or higher
- pip (Python package manager)
- Anthropic API key (from https://console.anthropic.com)

### Installation Steps

```bash
# 1. Navigate to your project directory
cd movie-ticket-booking-agent

# 2. Create a Python virtual environment
python -m venv venv

# 3. Activate virtual environment

# On Windows:
venv\Scripts\activate

# On macOS/Linux:
source venv/bin/activate

# 4. Install dependencies
pip install -r requirements.txt

# 5. Set up environment variables
# Copy the .env.example file
cp .env.example .env

# Open .env and add your Anthropic API key:
# ANTHROPIC_API_KEY=sk-ant-xxxxxxxxxxxxx
```

---

## Step 1: Set Up Backend API

### What We're Building

A REST API server that provides:
- Movie search and filtering
- Theatre information and filtering
- Show schedules and availability
- Booking creation and management

### Files Involved

```
backend/
  ├── models.py        - Data models
  ├── database.py      - In-memory database with sample data
  ├── routes.py        - API endpoints
  └── app.py           - Flask app setup
```

### How to Run

```bash
# From the project root directory
python -m backend.app
```

The API will start on `http://localhost:5000`

### Test the API

```bash
# In a new terminal, test the API

# Get all movies
curl http://localhost:5000/api/movies

# Search movies by genre
curl "http://localhost:5000/api/movies?genre=Action"

# Get theatres
curl http://localhost:5000/api/theatres

# Get shows
curl http://localhost:5000/api/shows?date=2024-03-25

# Check health
curl http://localhost:5000/api/health
```

### Understanding the Models

**Movie**
```python
{
    "id": 1,
    "title": "Mission Impossible: Dead Reckoning",
    "genre": "Action",
    "rating": 4.5,
    "duration": 163,  # in minutes
    "description": "...",
    "release_date": "2023-07-12"
}
```

**Theatre**
```python
{
    "id": 1,
    "name": "Downtown Cinema",
    "location": "Main Street",
    "city": "New York",
    "capacity": 250,
    "amenities": ["IMAX", "Dolby Surround", "Reclining Seats"]
}
```

**Show**
```python
{
    "id": 1,
    "movie_id": 1,
    "theatre_id": 1,
    "show_time": "7:00 PM",
    "date": "2024-03-25",
    "price": 18.00,
    "available_seats": 85
}
```

**Booking**
```python
{
    "id": "BK1000",
    "user_id": "user_123",
    "show_id": 1,
    "num_seats": 2,
    "seats": ["A1", "A2"],
    "total_price": 36.00,
    "booking_date": "2024-03-24T15:30:00",
    "status": "confirmed",
    "payment_status": "completed"
}
```

---

## Step 2: Create Mock Data

### What Happens Automatically

When you start the backend API, the `MockDatabase` class automatically:

1. **Initializes 10 sample movies**
   - Various genres (Action, Comedy, Drama, Sci-Fi, Horror)
   - Realistic ratings and descriptions
   - Duration and release dates

2. **Creates 5 sample theatres**
   - Different locations (New York, Los Angeles)
   - Various amenities (IMAX, 4DX, Premium seats)
   - Different capacities

3. **Generates show schedules**
   - Shows for tomorrow's date
   - Multiple time slots per theatre
   - Different pricing tiers
   - Seat availability

### Customize Mock Data

Edit `backend/database.py`:

```python
def _init_movies(self) -> Dict[int, Movie]:
    """Customize movie data here"""
    movies_data = [
        {
            "id": 1,
            "title": "Your Movie Title",
            "genre": "Your Genre",
            "rating": 4.5,
            "duration": 120,
            "description": "Movie description",
            "release_date": "2024-03-25"
        },
        # Add more movies...
    ]
    return {movie['id']: Movie(**movie) for movie in movies_data}
```

---

## Step 3: Build Claude Agent

### What We're Building

An intelligent agent that:
- Understands natural language queries
- Uses Claude's tool calling capability
- Interacts with the backend API
- Maintains conversation history
- Provides friendly responses

### Key Files

```
agent/
  ├── config.py       - Configuration and API keys
  ├── tools.py        - Tool definitions and API calls
  ├── prompts.py      - System prompts for Claude
  └── agent.py        - Main agent logic
```

### Tool Definitions

Your agent has access to 11 tools:

1. **search_movies** - Search movies by genre/rating
2. **get_movie_details** - Get specific movie info
3. **get_theatres** - List theatres by city
4. **get_theatre_details** - Get theatre info
5. **search_shows** - Find shows with filters
6. **get_show_details** - Get show information
7. **get_shows_for_movie** - Shows for a movie
8. **book_tickets** - Create a booking
9. **get_booking_details** - Get booking info
10. **get_user_bookings** - List user's bookings
11. **cancel_booking** - Cancel a booking

### How Tool Use Works

```
User: "Book 2 tickets for an action movie tomorrow"
        │
        ▼
┌─────────────────────────────────────────┐
│ Claude receives message & tools list    │
│ Analyzes: need to search for movies,    │
│ then find shows, then book tickets      │
└─────────────────────────────────────────┘
        │
        ├─→ Calls: search_movies(genre="Action")
        │   Gets: [list of action movies]
        │
        ├─→ Calls: get_shows_for_movie(movie_id=1)
        │   Gets: [available shows]
        │
        ├─→ Calls: book_tickets(user_id, show_id, 2 seats)
        │   Gets: {booking_id: "BK1000", ...}
        │
        ▼
Claude generates natural response to user
```

### Initialize and Use Agent

```python
from agent.agent import MovieBookingAgent

# Create agent for a user
agent = MovieBookingAgent(user_id="user_123")

# Have a conversation
response = agent.chat("I want to watch an action movie")
print(response)

# Agent handles tool calls automatically
response = agent.chat("Book 2 tickets please")
print(response)

# View conversation history
history = agent.get_history()
```

---

## Step 4: Test the Integration

### Test 1: Simple CLI Interaction

```bash
# Run the interactive agent
python agent/agent.py
```

This starts an interactive loop where you can chat with the agent.

### Test 2: Run Simple Example

```bash
# Run predefined conversation example
python examples/simple_booking.py
```

This demonstrates a complete booking flow.

### Test 3: Run Advanced Examples

```bash
# Run advanced scenarios
python examples/advanced_agent.py
```

This shows complex multi-turn conversations with filtering.

### Test 4: Complete Flow

**Terminal 1: Start Backend API**
```bash
python -m backend.app
# Output: Running on http://localhost:5000
```

**Terminal 2: Run Agent**
```bash
python agent/agent.py

# Then in the chat:
# You: I want a drama movie tomorrow evening
# [Chat continues...]
```

### Test 5: Web UI (Optional)

```bash
# Make sure backend is running on port 5000

# In a new terminal, start web UI
python examples/web_ui.py

# Open browser to http://localhost:5001
```

---

## Step 5: Deploy & Scale

### Moving Beyond Mock Data

For production, replace the mock database:

```python
# In backend/database.py, instead of MockDatabase:

class PostgresDatabase:
    """Real PostgreSQL database"""
    def __init__(self, connection_string):
        self.engine = create_engine(connection_string)
    
    def get_all_movies(self):
        # Query database
        pass
```

### Production Checklist

- [ ] Set up real database (PostgreSQL, MongoDB, etc.)
- [ ] Implement user authentication (JWT, OAuth)
- [ ] Add payment processing (Stripe, PayPal)
- [ ] Set up email notifications
- [ ] Add comprehensive logging
- [ ] Implement rate limiting
- [ ] Set up monitoring and alerts
- [ ] Add caching (Redis) for performance
- [ ] Deploy to cloud (AWS, Azure, GCP)
- [ ] Set up CI/CD pipeline

### Example Production Changes

```python
# 1. Add authentication middleware
@app.before_request
def verify_token():
    token = request.headers.get('Authorization')
    # Verify JWT token
    pass

# 2. Add error handling
try:
    booking = create_booking(...)
except DatabaseError as e:
    logger.error(f"Booking failed: {e}")
    return {"error": "Booking failed"}, 500

# 3. Add rate limiting
@limiter.limit("100 per hour")
@api_bp.route('/bookings', methods=['POST'])
def create_booking():
    pass

# 4. Add analytics tracking
def track_booking(booking_id, user_id):
    analytics.track("booking_created", {
        "booking_id": booking_id,
        "user_id": user_id,
        "timestamp": datetime.now()
    })
```

### Cloud Deployment

**AWS Example:**
```bash
# Package the app
pip freeze > requirements.txt
zip -r app.zip . -x "venv/*" ".git/*"

# Deploy to Lambda + API Gateway
# Set environment variables for ANTHROPIC_API_KEY
# Database: RDS PostgreSQL
```

**Docker:**
```dockerfile
FROM python:3.10
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "-m", "backend.app"]
```

---

## 📚 Example Conversations

### Example 1: Simple Movie Search

```
User: "Show me some action movies"

Claude: I'll search our catalog for action movies.
[Calls: search_movies(genre="Action")]

Claude: Great! I found several action movies available:

1. **Mission Impossible: Dead Reckoning** (Rating: 4.5/5)
   - Duration: 2h 43min
   - Overview: Ethan Hunt faces his most dangerous mission yet

2. **John Wick: Chapter 4** (Rating: 4.4/5)
   - Duration: 2h 49min
   - Overview: John takes on his most formidable enemies

Would you like to book a ticket for any of these?
```

### Example 2: Location-Based Search with Booking

```
User: "I want to watch Oppenheimer tomorrow in New York around 8 PM, budget $20"

Claude: Let me find shows for Oppenheimer in New York theatres.
[Calls: get_shows_for_movie(movie_id=4)]
[Calls: search_shows(movie_id=4, max_price=20, date="2024-03-25")]

Claude: I found some great options for you:

🎬 **Oppenheimer** (Duration: 3h)
━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. Downtown Cinema - 7:45 PM - $18.00 - 82 seats available
2. Metro Theatre - 8:15 PM - $20.00 - 45 seats available
3. Central Plex - 8:00 PM - $15.00 - 120 seats available

Which one do you prefer? How many tickets do you need?

User: "Book 2 tickets for Metro Theatre at 8:15 PM, seats A5 and A6"

Claude: Perfect! Let me complete your booking.
[Calls: book_tickets(user_id, show_id=45, num_seats=2, seats=["A5", "A6"])]

Claude: ✅ **Booking Confirmed!**

Confirmation Number: **BK1025**
Movie: Oppenheimer
Theatre: Metro Theatre, 5th Avenue, New York
Date: March 25, 2024
Time: 8:15 PM
Seats: A5, A6
Total Price: **$40.00**

Your confirmation has been saved. Enjoy the movie!
```

### Example 3: Booking Management

```
User: "Show me my bookings"

Claude: Let me retrieve your bookings.
[Calls: get_user_bookings(user_id="user_123")]

Claude: Here are your bookings:

📌 **Active Bookings**
1. Oppenheimer @ Metro Theatre
   - Date: March 25, 2024 at 8:15 PM
   - Seats: A5, A6
   - Confirmation: BK1025

2. Inside Out 2 @ Downtown Cinema
   - Date: March 26, 2024 at 3:00 PM
   - Seats: C2, C3
   - Confirmation: BK1026

Would you like to modify or cancel any booking?

User: "Cancel booking BK1025"

Claude: I'll cancel that for you.
[Calls: cancel_booking(booking_id="BK1025")]

Claude: ✅ Booking cancelled successfully!

Booking: BK1025
Movie: Oppenheimer
Refund: $40.00

Your refund will be processed within 3-5 business days.
```

---

## 🔧 API Reference

### Movies Endpoint

```
GET /api/movies
Query Parameters:
  - genre: (optional) Filter by genre
  - min_rating: (optional) Minimum rating

Response:
{
    "success": true,
    "count": 3,
    "movies": [
        {
            "id": 1,
            "title": "...",
            "genre": "...",
            "rating": 4.5,
            ...
        }
    ]
}
```

### Shows Endpoint

```
GET /api/shows
Query Parameters:
  - movie_id: (optional) Filter by movie
  - theatre_id: (optional) Filter by theatre
  - date: (optional) Filter by date (YYYY-MM-DD)
  - max_price: (optional) Maximum price

Response:
{
    "success": true,
    "count": 5,
    "shows": [
        {
            "id": 1,
            "movie_id": 1,
            "theatre_id": 1,
            "show_time": "7:00 PM",
            "date": "2024-03-25",
            "price": 18.00,
            "available_seats": 85,
            "movie_title": "...",
            "theatre_name": "..."
        }
    ]
}
```

### Bookings Endpoint

```
POST /api/bookings
Request Body:
{
    "user_id": "user_123",
    "show_id": 1,
    "num_seats": 2,
    "seats": ["A1", "A2"]
}

Response:
{
    "success": true,
    "booking": {
        "id": "BK1000",
        "user_id": "user_123",
        "show_id": 1,
        "num_seats": 2,
        "seats": ["A1", "A2"],
        "total_price": 36.00,
        "status": "confirmed",
        "payment_status": "completed"
    }
}
```

---

## ❓ Troubleshooting

### Issue: "ANTHROPIC_API_KEY not found"

**Solution:** Make sure you've set up the `.env` file:
```bash
cp .env.example .env
# Edit .env and add your API key
```

### Issue: Backend API not responding

**Solution:** Make sure the backend is running:
```bash
python -m backend.app
# Check: http://localhost:5000/api/health
```

### Issue: Tool calls not working

**Solution:** Verify tool definitions in `agent/tools.py` match the backend API endpoints.

### Issue: Booking returns "insufficient seats"

**Solution:** Check available seats with `get_show_details()` before booking.

---

## 📖 Further Learning

- [Claude Tool Use Documentation](https://docs.anthropic.com/en/docs/build-a-chatbot-with-tool-use)
- [Claude API Reference](https://docs.anthropic.com/en/api)
- [Flask Documentation](https://flask.palletsprojects.com/)

---

## Next Steps

1. ✅ Run the backend API
2. ✅ Test with simple examples
3. ✅ Build your own conversation flows
4. ✅ Integrate with real database
5. ✅ Deploy to production
6. ✅ Add payment processing
7. ✅ Implement user authentication

---

Good luck with your movie booking agent! 🎬
