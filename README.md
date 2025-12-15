# Scopreneur Initiative - Backend API

Complete backend API for the Scopreneur educational platform with authentication, booking system, payments, and admin management.

---

## 🚀 Quick Start

### Prerequisites
- Node.js (v14 or higher)
- MongoDB (local or MongoDB Atlas)
- Paystack account (for payments)

### Installation

1. **Install Dependencies**
```bash
npm install
```

2. **Setup Environment Variables**
   - Copy `.env.example` to `.env`
   - Update all values with your credentials

3. **Start MongoDB** (if using local)
```bash
mongod
```

4. **Run the Server**
```bash
# Development mode (with auto-restart)
npm run dev

# Production mode
npm start
```

Server will run on `http://localhost:5000`

---

## 📁 Project Structure

```
scopreneur-backend/
├── server.js           # Main application file
├── package.json        # Dependencies
├── .env               # Environment variables (create from .env.example)
└── README.md          # Documentation
```

---

## 🔐 Authentication

All protected routes require a JWT token in the Authorization header:
```
Authorization: Bearer <your-jwt-token>
```

---

## 📚 API Endpoints

### **Authentication Routes**

#### Register Student/Parent
```http
POST /api/auth/register
Content-Type: application/json

{
  "fullName": "John Doe",
  "email": "john@example.com",
  "password": "securePassword123",
  "phone": "+2348012345678"
}
```

**Response:**
```json
{
  "message": "Registration successful!",
  "token": "jwt-token-here",
  "user": {
    "id": "user-id",
    "fullName": "John Doe",
    "email": "john@example.com",
    "role": "student"
  }
}
```

#### Register Tutor
```http
POST /api/auth/register-tutor
Content-Type: application/json

{
  "fullName": "Jane Smith",
  "email": "jane@example.com",
  "password": "securePassword123",
  "phone": "+2348012345678",
  "bio": "Experienced mathematics tutor with 5+ years...",
  "qualifications": "BSc Mathematics, M.Ed",
  "experience": "5 years teaching secondary school students",
  "categories": ["academic", "digital-skills"],
  "subjects": ["Mathematics", "Python Programming"],
  "hourlyRate": 5000,
  "sessionTypes": ["in-person", "online"]
}
```

**Response:**
```json
{
  "message": "Tutor registration submitted! Awaiting admin approval.",
  "user": {
    "id": "user-id",
    "fullName": "Jane Smith",
    "email": "jane@example.com",
    "role": "tutor",
    "isApproved": false
  }
}
```

#### Login
```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "john@example.com",
  "password": "securePassword123"
}
```

**Response:**
```json
{
  "message": "Login successful!",
  "token": "jwt-token-here",
  "user": {
    "id": "user-id",
    "fullName": "John Doe",
    "email": "john@example.com",
    "role": "student",
    "isApproved": true
  }
}
```

#### Get Current User
```http
GET /api/auth/me
Authorization: Bearer <token>
```

---

### **Tutor Routes**

#### Get All Tutors (Public)
```http
GET /api/tutors?category=academic&subject=Mathematics&minRate=3000&maxRate=10000
```

**Response:**
```json
{
  "tutors": [
    {
      "_id": "tutor-profile-id",
      "userId": {
        "_id": "user-id",
        "fullName": "Jane Smith",
        "email": "jane@example.com",
        "phone": "+2348012345678"
      },
      "bio": "Experienced mathematics tutor...",
      "qualifications": "BSc Mathematics, M.Ed",
      "experience": "5 years",
      "categories": ["academic"],
      "subjects": ["Mathematics", "English"],
      "hourlyRate": 5000,
      "sessionTypes": ["in-person", "online"],
      "rating": 4.8,
      "totalRatings": 25,
      "totalSessions": 150,
      "isActive": true
    }
  ]
}
```

#### Get Single Tutor
```http
GET /api/tutors/:userId
```

#### Update Tutor Profile (Protected - Tutor Only)
```http
PUT /api/tutors/profile
Authorization: Bearer <token>
Content-Type: application/json

{
  "bio": "Updated bio...",
  "hourlyRate": 6000,
  "subjects": ["Mathematics", "Physics", "Python"],
  "availability": {
    "monday": [{ "start": "09:00", "end": "17:00" }],
    "tuesday": [{ "start": "09:00", "end": "17:00" }]
  }
}
```

---

### **Booking Routes**

#### Create Booking (Protected - Student)
```http
POST /api/bookings
Authorization: Bearer <token>
Content-Type: application/json

{
  "tutorId": "tutor-user-id",
  "category": "academic",
  "subject": "Mathematics",
  "sessionType": "online",
  "date": "2025-10-15",
  "time": "14:00",
  "duration": 2,
  "notes": "Need help with calculus"
}
```

**Response:**
```json
{
  "message": "Booking created! Proceed to payment.",
  "booking": {
    "_id": "booking-id",
    "studentId": "student-id",
    "tutorId": "tutor-id",
    "category": "academic",
    "subject": "Mathematics",
    "sessionType": "online",
    "date": "2025-10-15T00:00:00.000Z",
    "time": "14:00",
    "duration": 2,
    "totalAmount": 10000,
    "commission": 1500,
    "tutorEarnings": 8500,
    "status": "pending",
    "paymentStatus": "pending"
  }
}
```

#### Get My Bookings (Protected)
```http
GET /api/bookings/my-bookings
Authorization: Bearer <token>
```

#### Get Single Booking (Protected)
```http
GET /api/bookings/:bookingId
Authorization: Bearer <token>
```

#### Update Booking Status (Protected - Tutor/Admin)
```http
PUT /api/bookings/:bookingId/status
Authorization: Bearer <token>
Content-Type: application/json

{
  "status": "confirmed" // or "completed", "cancelled"
}
```

---

### **Payment Routes**

#### Initialize Payment (Protected - Student)
```http
POST /api/payments/initialize
Authorization: Bearer <token>
Content-Type: application/json

{
  "bookingId": "booking-id"
}
```

**Response:**
```json
{
  "message": "Payment initialized",
  "reference": "SCO-1234567890-booking-id",
  "amount": 10000,
  "paymentUrl": "https://paystack.com/pay/reference"
}
```

#### Verify Payment (Webhook)
```http
POST /api/payments/verify
Content-Type: application/json

{
  "reference": "SCO-1234567890-booking-id"
}
```

---

### **Rating Routes**

#### Create Rating (Protected - Student)
```http
POST /api/ratings
Authorization: Bearer <token>
Content-Type: application/json

{
  "bookingId": "booking-id",
  "rating": 5,
  "review": "Excellent tutor! Very helpful and patient."
}
```

#### Get Tutor Ratings (Public)
```http
GET /api/ratings/tutor/:tutorId
```

**Response:**
```json
{
  "ratings": [
    {
      "_id": "rating-id",
      "bookingId": "booking-id",
      "tutorId": "tutor-id",
      "studentId": {
        "_id": "student-id",
        "fullName": "John Doe"
      },
      "rating": 5,
      "review": "Excellent tutor!",
      "createdAt": "2025-10-11T10:00:00.000Z"
    }
  ]
}
```

---

### **Admin Routes** (Protected - Admin Only)

#### Get Pending Tutors
```http
GET /api/admin/pending-tutors
Authorization: Bearer <admin-token>
```

#### Approve/Reject Tutor
```http
PUT /api/admin/tutors/:tutorId/approval
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "approved": true // or false to reject
}
```

#### Get All Bookings
```http
GET /api/admin/bookings
Authorization: Bearer <admin-token>
```

#### Get Analytics
```http
GET /api/admin/analytics
Authorization: Bearer <admin-token>
```

**Response:**
```json
{
  "analytics": {
    "totalUsers": 150,
    "totalTutors": 25,
    "totalBookings": 500,
    "totalRevenue": 750000
  }
}
```

---

## 🗄️ Database Models

### User
- fullName, email, password (hashed), phone
- role: student | tutor | admin
- isVerified, isApproved

### TutorProfile
- userId (ref: User)
- bio, qualifications, experience
- categories, subjects, hourlyRate
- sessionTypes, availability
- rating, totalRatings, totalSessions
- subscriptionStatus, subscriptionExpiry

### Booking
- studentId, tutorId
- category, subject, sessionType
- date, time, duration
- totalAmount, commission, tutorEarnings
- status: pending | confirmed | completed | cancelled
- paymentStatus: pending | paid | refunded

### Rating
- bookingId, tutorId, studentId
- rating (1-5), review

### Payment
- bookingId, userId
- amount, commission, tutorEarnings
- reference, status
- paystackResponse

### Subscription
- tutorId, amount
- startDate, endDate
- paymentReference, status

---

## 🔧 Configuration

### Commission Rate
Set in `.env`: `COMMISSION_RATE=0.15` (15%)

### Tutor Subscription
Set in `.env`: `TUTOR_SUBSCRIPTION_FEE=5000` (₦5,000/month)

---

## 🎯 Next Steps

1. **Create Admin Account**
   - Manually create an admin user in MongoDB:
   ```javascript
   {
     "fullName": "Admin User",
     "email": "admin@scopreneur.com",
     "password": "<hashed-password>",
     "phone": "+2348000000000",
     "role": "admin",
     "isVerified": true,
     "isApproved": true
   }
   ```

2. **Integrate Paystack**
   - Get API keys from https://paystack.com
   - Add to `.env` file
   - Implement webhook for payment verification

3. **Deploy Backend**
   - Railway.app (recommended)
   - Render.com
   - Heroku
   - DigitalOcean

4. **Setup MongoDB Atlas** (for production)
   - Create cluster at https://mongodb.com/atlas
   - Get connection string
   - Update `MONGODB_URI` in `.env`

---

## 🐛 Common Issues

### MongoDB Connection Error
- Make sure MongoDB is running
- Check connection string in `.env`

### JWT Token Error
- Ensure `JWT_SECRET` is set in `.env`
- Token format: `Bearer <token>`

### CORS Error
- Update `FRONTEND_URL` in `.env`
- Check CORS configuration in `server.js`

---

## 📞 Support

For issues or questions:
- Email: scopreneurinitiative@gmail.com

---

## 📄 License

© 2025 Scopreneur Initiative. All rights reserved.