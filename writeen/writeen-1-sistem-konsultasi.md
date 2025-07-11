# Writeen - Sistem Jadwal Konsultasi
## Dokumentasi Lengkap Feature 1

### 📋 Daftar Isi
1. [Overview Sistem](#overview-sistem)
2. [Arsitektur Sistem](#arsitektur-sistem)
3. [User Flow](#user-flow)
4. [Database Schema](#database-schema)
5. [API Specification](#api-specification)
6. [Frontend Implementation](#frontend-implementation)
7. [Backend Implementation](#backend-implementation)
8. [Integrasi Kalender](#integrasi-kalender)
9. [Sistem Reminder](#sistem-reminder)
10. [Testing Strategy](#testing-strategy)
11. [Deployment](#deployment)

---

## 🎯 Overview Sistem

### Tujuan
Membangun sistem jadwal konsultasi yang memungkinkan mahasiswa dan tutor untuk:
- Membuat, mengelola, dan mengatur jadwal konsultasi
- Integrasi dengan kalender internal aplikasi
- Sistem reminder otomatis untuk semua pihak
- Manajemen konflik jadwal dan validasi waktu

> **📝 Catatan Phase 1**: Pada fase ini, "konsultasi" dilakukan melalui **chat room/session** di dalam aplikasi. Fitur video call akan dikembangkan pada fase berikutnya.

### Target Users
- **Mahasiswa**: Dapat membuat request konsultasi, melihat jadwal tutor, dan mengelola appointment
- **Tutor**: Dapat mengatur ketersediaan, menerima/menolak request, dan mengelola jadwal konsultasi

### Key Features
1. **Manajemen Jadwal Konsultasi**
   - Create, Read, Update, Delete (CRUD) konsultasi
   - Validasi konflik jadwal
   - Status tracking (pending, confirmed, cancelled, completed)
   
2. **Integrasi Kalender Internal**
   - Calendar view dengan table_calendar
   - Event markers dan color coding
   - Multiple view modes (month, week, agenda)
   
3. **Sistem Reminder Otomatis**
   - Push notifications
   - Email notifications
   - In-app notifications
   - Customizable reminder timing

---

## 🏗️ Arsitektur Sistem

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    📱 FLUTTER FRONTEND                          │
├─────────────────────────────────────────────────────────────────┤
│  Schedule Screen    │  Calendar Widget   │  Notification       │
│  - List View        │  - Event Display   │  - Push Notifications │
│  - Create Form      │  - Time Slots      │  - In-App Alerts    │
│  - Edit/Cancel      │  - Availability    │  - Email Reminders  │
└─────────────────────────────────────────────────────────────────┘
                              │
                         HTTP/REST API
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    🔧 GOLANG BACKEND                            │
├─────────────────────────────────────────────────────────────────┤
│  Authentication     │  Schedule Service  │  Notification       │
│  - JWT Middleware   │  - CRUD Operations │  - Email Service    │
│  - User Validation  │  - Conflict Check  │  - Push Service     │
│  - Role Management  │  - Availability    │  - Reminder Jobs    │
└─────────────────────────────────────────────────────────────────┘
                              │
                         MongoDB Driver
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    🗄️ MONGODB DATABASE                          │
├─────────────────────────────────────────────────────────────────┤
│  Collections:                                                   │
│  - consultations    │  - users           │  - notifications    │
│  - availability     │  - schedules       │  - reminders        │
│  - conflicts        │  - time_slots      │  - templates        │
└─────────────────────────────────────────────────────────────────┘
```

### Technology Stack
- **Frontend**: Flutter + Dart + GetX
- **Backend**: Golang + Gin Framework
- **Database**: MongoDB
- **Calendar**: table_calendar package
- **Notifications**: In-app notifications via WebSocket
- **Email**: SMTP (Gmail/SendGrid)
- **Scheduling**: Cron jobs dengan golang

---

## 👥 User Flow

### 1. Student Flow - Membuat Konsultasi

```
🎓 STUDENT JOURNEY
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Login App     │───▶│  Browse Tutors  │───▶│  Select Tutor   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Get Reminder  │◀───│  Wait Approval  │◀───│  Submit Request │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Join Chat      │───▶│  Consultation   │───▶│  Provide Rating │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

**Detailed Steps:**
1. **Login & Dashboard**
   - Student login dengan email/password
   - Melihat dashboard dengan upcoming consultations
   - Quick access ke schedule new consultation

2. **Browse Available Tutors**
   - Filter by subject/expertise
   - View tutor profiles dan rating
   - Check availability calendar

3. **Select Time Slot**
   - View tutor's available time slots
   - Select preferred date and time
   - See conflict warnings if any

4. **Submit Consultation Request**
   - Fill consultation details (topic, duration, notes)
   - Upload supporting documents if needed
   - Submit for tutor approval

5. **Wait for Approval**
   - Receive notification when tutor responds
   - Auto-reminder if no response within 24 hours
   - Option to cancel pending request

6. **Confirmed Consultation**
   - Receive confirmation with chat session details
   - Add to calendar automatically
   - Get reminders (24h, 1h, 15min before)

### 2. Tutor Flow - Mengelola Konsultasi

```
👨‍🏫 TUTOR JOURNEY
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Set Available │───▶│  Receive Request│───▶│  Review Request │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Get Reminder   │◀───│  Confirmed Slot │◀───│ Accept/Decline  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Conduct Session│───▶│  Mark Complete  │───▶│  Receive Rating │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

**Detailed Steps:**
1. **Set Availability**
   - Configure working hours and days
   - Set consultation duration preferences
   - Block specific dates/times for holidays

2. **Receive Consultation Requests**
   - Get push notification for new requests
   - Review student details and consultation topic
   - Check for schedule conflicts

3. **Accept/Decline Requests**
   - Approve with confirmation message
   - Decline with reason and suggest alternatives
   - Automatically update calendar

4. **Manage Upcoming Consultations**
   - View today's schedule
   - Receive reminders before each session
   - Access student materials and notes

5. **Conduct & Complete Sessions**
   - Mark consultation as completed
   - Add session notes and feedback
   - Rate student participation

---

## 📊 Database Schema

### 1. Users Collection
```json
{
  "_id": "ObjectId",
  "email": "string",
  "password": "string (hashed)",
  "role": "student|tutor",
  "profile": {
    "firstName": "string",
    "lastName": "string",
    "phoneNumber": "string",
    "avatar": "string",
    "bio": "string",
    "specialization": ["string"], // for tutors
    "university": "string", // for students
    "major": "string" // for students
  },
  "preferences": {
    "timezone": "string",
    "language": "string",
    "notifications": {
      "email": true,
      "push": true,
      "sms": false
    }
  },
  "createdAt": "Date",
  "updatedAt": "Date"
}
```

### 2. Consultations Collection
```json
{
  "_id": "ObjectId",
  "studentId": "ObjectId",
  "tutorId": "ObjectId",
  "title": "string",
  "description": "string",
  "subject": "string",
  "status": "pending|confirmed|cancelled|completed|no_show",
  "schedule": {
    "startTime": "Date",
    "endTime": "Date",
    "duration": "number (minutes)",
    "timezone": "string"
  },
  "meeting": {
    "type": "chat", // Phase 1: chat only, video calls in next phase
    "chatRoomId": "string",
    "platform": "writeen_chat", // internal chat system
    "sessionUrl": "string", // URL to chat room
    "scheduledDuration": "number", // minutes
    "location": "string" // optional for future in-person meetings
  },
  "documents": [
    {
      "name": "string",
      "url": "string",
      "type": "string",
      "uploadedBy": "ObjectId",
      "uploadedAt": "Date"
    }
  ],
  "notes": {
    "studentNotes": "string",
    "tutorNotes": "string",
    "sessionSummary": "string"
  },
  "rating": {
    "studentRating": "number",
    "tutorRating": "number",
    "feedback": "string"
  },
  "reminders": [
    {
      "type": "email|push|sms",
      "scheduledAt": "Date",
      "sent": "boolean",
      "sentAt": "Date"
    }
  ],
  "createdAt": "Date",
  "updatedAt": "Date",
  "cancelledAt": "Date",
  "cancellationReason": "string"
}
```

### 3. Availability Collection
```json
{
  "_id": "ObjectId",
  "tutorId": "ObjectId",
  "type": "weekly|specific_date|holiday",
  "weeklySchedule": {
    "monday": [
      {
        "startTime": "09:00",
        "endTime": "12:00",
        "duration": 60, // consultation duration in minutes
        "maxConsultations": 3
      }
    ],
    "tuesday": [...],
    // ... other days
  },
  "specificDates": [
    {
      "date": "Date",
      "available": "boolean",
      "timeSlots": [
        {
          "startTime": "Time",
          "endTime": "Time",
          "booked": "boolean",
          "consultationId": "ObjectId"
        }
      ]
    }
  ],
  "holidays": [
    {
      "startDate": "Date",
      "endDate": "Date",
      "reason": "string"
    }
  ],
  "createdAt": "Date",
  "updatedAt": "Date"
}
```

### 4. Notifications Collection
```json
{
  "_id": "ObjectId",
  "userId": "ObjectId",
  "consultationId": "ObjectId",
  "type": "reminder|status_update|system",
  "subType": "24h_reminder|1h_reminder|15min_reminder|confirmed|cancelled",
  "title": "string",
  "message": "string",
  "channels": ["email", "push", "sms"],
  "status": "pending|sent|failed",
  "scheduledAt": "Date",
  "sentAt": "Date",
  "readAt": "Date",
  "data": {
    "consultationDetails": "object",
    "actionUrl": "string"
  },
  "createdAt": "Date"
}
```

### 5. Time Slots Collection (untuk optimasi query)
```json
{
  "_id": "ObjectId",
  "tutorId": "ObjectId",
  "date": "Date",
  "startTime": "Date",
  "endTime": "Date",
  "status": "available|booked|blocked",
  "consultationId": "ObjectId", // jika booked
  "createdAt": "Date"
}
```

---

## 🔌 API Specification

### Base URL
```
Development: http://localhost:8080
Production: https://api.writeen.com
```

### Authentication
```
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

### 1. Authentication Endpoints

#### POST /api/auth/register
```json
Request:
{
  "email": "student@university.ac.id",
  "password": "securepassword",
  "role": "student",
  "profile": {
    "firstName": "John",
    "lastName": "Doe",
    "phoneNumber": "+6281234567890",
    "university": "University of Indonesia",
    "major": "Computer Science"
  }
}

Response:
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "user": {
      "id": "user_id",
      "email": "student@university.ac.id",
      "role": "student",
      "profile": {...}
    },
    "token": "jwt_token"
  }
}
```

#### POST /api/auth/login
```json
Request:
{
  "email": "student@university.ac.id",
  "password": "securepassword"
}

Response:
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": {...},
    "token": "jwt_token",
    "expiresAt": "2024-12-31T23:59:59Z"
  }
}
```

### 2. Schedule/Consultation Endpoints

#### GET /api/consultations
```json
Query Parameters:
- page: number (default: 1)
- limit: number (default: 10)
- status: string (pending|confirmed|cancelled|completed)
- startDate: date
- endDate: date
- tutorId: string
- studentId: string

Response:
{
  "success": true,
  "data": {
    "consultations": [
      {
        "id": "consultation_id",
        "title": "Thesis Discussion",
        "description": "Need help with chapter 3",
        "status": "confirmed",
        "schedule": {
          "startTime": "2024-01-15T10:00:00Z",
          "endTime": "2024-01-15T11:00:00Z",
          "duration": 60
        },
        "tutor": {
          "id": "tutor_id",
          "name": "Dr. Jane Smith",
          "specialization": ["Computer Science", "AI"]
        },
        "student": {
          "id": "student_id",
          "name": "John Doe",
          "university": "University of Indonesia"
        },
        "meeting": {
          "type": "chat",
          "platform": "writeen_chat",
          "sessionUrl": "/chat/room/abc123",
          "chatRoomId": "consultation_abc123"
        }
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 25,
      "totalPages": 3
    }
  }
}
```

#### POST /api/consultations
```json
Request:
{
  "tutorId": "tutor_id",
  "title": "Thesis Discussion",
  "description": "Need help with chapter 3 methodology",
  "subject": "Computer Science",
  "schedule": {
    "startTime": "2024-01-15T10:00:00Z",
    "endTime": "2024-01-15T11:00:00Z",
    "timezone": "Asia/Jakarta"
  },
  "meeting": {
    "type": "chat",
    "platform": "writeen_chat"
  },
  "studentNotes": "Please review attached document"
}

Response:
{
  "success": true,
  "message": "Consultation request created successfully",
  "data": {
    "consultation": {
      "id": "consultation_id",
      "status": "pending",
      "schedule": {...},
      "tutor": {...},
      "student": {...}
    }
  }
}
```

#### PUT /api/consultations/:id/status
```json
Request:
{
  "status": "confirmed|cancelled",
  "reason": "Schedule conflict", // required for cancellation
  "notes": "Looking forward to the session"
}

Response:
{
  "success": true,
  "message": "Consultation status updated successfully",
  "data": {
    "consultation": {
      "id": "consultation_id",
      "status": "confirmed",
      "updatedAt": "2024-01-10T15:30:00Z"
    }
  }
}
```

### 3. Availability Endpoints

#### GET /api/availability/:tutorId
```json
Query Parameters:
- startDate: date
- endDate: date
- duration: number (minutes)

Response:
{
  "success": true,
  "data": {
    "availability": [
      {
        "date": "2024-01-15",
        "slots": [
          {
            "startTime": "09:00",
            "endTime": "10:00",
            "available": true,
            "consultationId": null
          },
          {
            "startTime": "10:00",
            "endTime": "11:00",
            "available": false,
            "consultationId": "consultation_id"
          }
        ]
      }
    ],
    "tutorInfo": {
      "id": "tutor_id",
      "name": "Dr. Jane Smith",
      "specialization": ["Computer Science"],
      "rating": 4.8
    }
  }
}
```

#### POST /api/availability
```json
Request:
{
  "type": "weekly",
  "weeklySchedule": {
    "monday": [
      {
        "startTime": "09:00",
        "endTime": "12:00",
        "duration": 60,
        "maxConsultations": 3
      }
    ],
    "tuesday": [...]
  }
}

Response:
{
  "success": true,
  "message": "Availability updated successfully",
  "data": {
    "availability": {...}
  }
}
```

### 4. Notification Endpoints

#### GET /api/notifications
```json
Query Parameters:
- page: number
- limit: number
- read: boolean
- type: string

Response:
{
  "success": true,
  "data": {
    "notifications": [
      {
        "id": "notification_id",
        "type": "reminder",
        "title": "Consultation Reminder",
        "message": "Your consultation with Dr. Jane Smith starts in 1 hour",
        "readAt": null,
        "createdAt": "2024-01-15T08:00:00Z"
      }
    ],
    "unreadCount": 5
  }
}
```

#### PUT /api/notifications/:id/read
```json
Response:
{
  "success": true,
  "message": "Notification marked as read"
}
```

---

## 📱 Frontend Implementation

### 1. Project Structure
```
lib/
├── app/
│   ├── controllers/
│   │   ├── schedule_controller.dart
│   │   ├── consultation_controller.dart
│   │   └── notification_controller.dart
│   ├── models/
│   │   ├── consultation.dart
│   │   ├── availability.dart
│   │   └── notification.dart
│   ├── services/
│   │   ├── api_service.dart
│   │   ├── notification_service.dart
│   │   └── calendar_service.dart
│   └── views/
│       ├── schedule/
│       │   ├── schedule_view.dart
│       │   ├── consultation_form.dart
│       │   └── consultation_detail.dart
│       └── components/
│           ├── calendar_widget.dart
│           ├── time_slot_picker.dart
│           └── consultation_card.dart
```

### 2. Core Models

#### Consultation Model
```dart
class Consultation {
  final String id;
  final String studentId;
  final String tutorId;
  final String title;
  final String description;
  final String subject;
  final ConsultationStatus status;
  final Schedule schedule;
  final Meeting meeting;
  final List<Document> documents;
  final Notes notes;
  final Rating? rating;
  final DateTime createdAt;
  final DateTime updatedAt;

  Consultation({
    required this.id,
    required this.studentId,
    required this.tutorId,
    required this.title,
    required this.description,
    required this.subject,
    required this.status,
    required this.schedule,
    required this.meeting,
    this.documents = const [],
    required this.notes,
    this.rating,
    required this.createdAt,
    required this.updatedAt,
  });

  factory Consultation.fromJson(Map<String, dynamic> json) {
    return Consultation(
      id: json['id'],
      studentId: json['studentId'],
      tutorId: json['tutorId'],
      title: json['title'],
      description: json['description'],
      subject: json['subject'],
      status: ConsultationStatus.values.firstWhere(
        (e) => e.name == json['status'],
      ),
      schedule: Schedule.fromJson(json['schedule']),
      meeting: Meeting.fromJson(json['meeting']),
      documents: (json['documents'] as List<dynamic>?)
          ?.map((doc) => Document.fromJson(doc))
          .toList() ?? [],
      notes: Notes.fromJson(json['notes']),
      rating: json['rating'] != null ? Rating.fromJson(json['rating']) : null,
      createdAt: DateTime.parse(json['createdAt']),
      updatedAt: DateTime.parse(json['updatedAt']),
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'id': id,
      'studentId': studentId,
      'tutorId': tutorId,
      'title': title,
      'description': description,
      'subject': subject,
      'status': status.name,
      'schedule': schedule.toJson(),
      'meeting': meeting.toJson(),
      'documents': documents.map((doc) => doc.toJson()).toList(),
      'notes': notes.toJson(),
      'rating': rating?.toJson(),
      'createdAt': createdAt.toIso8601String(),
      'updatedAt': updatedAt.toIso8601String(),
    };
  }
}

enum ConsultationStatus {
  pending,
  confirmed,
  cancelled,
  completed,
  noShow
}

class Schedule {
  final DateTime startTime;
  final DateTime endTime;
  final int duration;
  final String timezone;

  Schedule({
    required this.startTime,
    required this.endTime,
    required this.duration,
    required this.timezone,
  });

  factory Schedule.fromJson(Map<String, dynamic> json) {
    return Schedule(
      startTime: DateTime.parse(json['startTime']),
      endTime: DateTime.parse(json['endTime']),
      duration: json['duration'],
      timezone: json['timezone'],
    );
  }

  Map<String, dynamic> toJson() {
    return {
      'startTime': startTime.toIso8601String(),
      'endTime': endTime.toIso8601String(),
      'duration': duration,
      'timezone': timezone,
    };
  }
}
```

### 3. Schedule Controller
```dart
class ScheduleController extends GetxController {
  final ApiService _apiService = Get.find<ApiService>();
  final NotificationService _notificationService = Get.find<NotificationService>();
  
  // Observable variables
  final RxList<Consultation> consultations = <Consultation>[].obs;
  final RxList<Consultation> todayConsultations = <Consultation>[].obs;
  final RxList<Consultation> upcomingConsultations = <Consultation>[].obs;
  final RxBool isLoading = false.obs;
  final RxString error = ''.obs;
  
  // Calendar variables
  final Rx<DateTime> selectedDate = DateTime.now().obs;
  final Rx<DateTime> focusedDate = DateTime.now().obs;
  final RxMap<DateTime, List<Consultation>> events = <DateTime, List<Consultation>>{}.obs;

  @override
  void onInit() {
    super.onInit();
    loadConsultations();
    setupNotificationListeners();
  }

  Future<void> loadConsultations() async {
    try {
      isLoading.value = true;
      error.value = '';
      
      final response = await _apiService.getConsultations();
      consultations.value = response.data;
      
      // Group consultations by date for calendar
      _groupConsultationsByDate();
      
      // Filter today's consultations
      _filterTodayConsultations();
      
      // Filter upcoming consultations
      _filterUpcomingConsultations();
      
    } catch (e) {
      error.value = e.toString();
      Get.snackbar('Error', 'Failed to load consultations: ${e.toString()}');
    } finally {
      isLoading.value = false;
    }
  }

  Future<void> createConsultation(Consultation consultation) async {
    try {
      isLoading.value = true;
      
      final response = await _apiService.createConsultation(consultation);
      consultations.add(response.data);
      
      // Refresh calendar events
      _groupConsultationsByDate();
      
      Get.snackbar('Success', 'Consultation request created successfully');
      Get.back(); // Close form
      
    } catch (e) {
      Get.snackbar('Error', 'Failed to create consultation: ${e.toString()}');
    } finally {
      isLoading.value = false;
    }
  }

  Future<void> updateConsultationStatus(String id, ConsultationStatus status, {String? reason}) async {
    try {
      isLoading.value = true;
      
      await _apiService.updateConsultationStatus(id, status, reason: reason);
      
      // Update local consultation
      final index = consultations.indexWhere((c) => c.id == id);
      if (index != -1) {
        consultations[index] = consultations[index].copyWith(
          status: status,
          updatedAt: DateTime.now(),
        );
      }
      
      // Refresh calendar events
      _groupConsultationsByDate();
      
      Get.snackbar('Success', 'Consultation status updated successfully');
      
    } catch (e) {
      Get.snackbar('Error', 'Failed to update consultation: ${e.toString()}');
    } finally {
      isLoading.value = false;
    }
  }

  void _groupConsultationsByDate() {
    events.clear();
    for (final consultation in consultations) {
      final date = DateTime(
        consultation.schedule.startTime.year,
        consultation.schedule.startTime.month,
        consultation.schedule.startTime.day,
      );
      
      if (events.containsKey(date)) {
        events[date]!.add(consultation);
      } else {
        events[date] = [consultation];
      }
    }
  }

  void _filterTodayConsultations() {
    final today = DateTime.now();
    final todayStart = DateTime(today.year, today.month, today.day);
    final todayEnd = todayStart.add(Duration(days: 1));
    
    todayConsultations.value = consultations.where((consultation) {
      final consultationDate = consultation.schedule.startTime;
      return consultationDate.isAfter(todayStart) && 
             consultationDate.isBefore(todayEnd);
    }).toList();
  }

  void _filterUpcomingConsultations() {
    final now = DateTime.now();
    upcomingConsultations.value = consultations.where((consultation) {
      return consultation.schedule.startTime.isAfter(now) &&
             consultation.status == ConsultationStatus.confirmed;
    }).toList();
    
    // Sort by date
    upcomingConsultations.sort((a, b) => 
      a.schedule.startTime.compareTo(b.schedule.startTime));
  }

  void setupNotificationListeners() {
    _notificationService.onNotificationReceived.listen((notification) {
      if (notification.type == NotificationType.consultationReminder) {
        // Show in-app notification
        Get.snackbar(
          'Reminder',
          notification.message,
          backgroundColor: Colors.blue,
          colorText: Colors.white,
        );
      }
    });
  }

  // Calendar event providers
  List<Consultation> getEventsForDay(DateTime day) {
    final normalizedDay = DateTime(day.year, day.month, day.day);
    return events[normalizedDay] ?? [];
  }

  bool hasEventsForDay(DateTime day) {
    return getEventsForDay(day).isNotEmpty;
  }

  void onDaySelected(DateTime selectedDay, DateTime focusedDay) {
    selectedDate.value = selectedDay;
    focusedDate.value = focusedDay;
  }
}
```

### 4. Calendar Widget Implementation
```dart
class CalendarWidget extends StatelessWidget {
  final ScheduleController controller = Get.find<ScheduleController>();

  @override
  Widget build(BuildContext context) {
    return Obx(() => Card(
      elevation: 2,
      child: TableCalendar<Consultation>(
        firstDay: DateTime.utc(2023, 1, 1),
        lastDay: DateTime.utc(2025, 12, 31),
        focusedDay: controller.focusedDate.value,
        selectedDayPredicate: (day) {
          return isSameDay(controller.selectedDate.value, day);
        },
        eventLoader: controller.getEventsForDay,
        startingDayOfWeek: StartingDayOfWeek.monday,
        calendarStyle: CalendarStyle(
          outsideDaysVisible: false,
          weekendTextStyle: TextStyle(color: Colors.red),
          selectedDecoration: BoxDecoration(
            color: AppColors.primaryBlue,
            shape: BoxShape.circle,
          ),
          todayDecoration: BoxDecoration(
            color: AppColors.primaryLight,
            shape: BoxShape.circle,
          ),
          markerDecoration: BoxDecoration(
            color: AppColors.accent,
            shape: BoxShape.circle,
          ),
        ),
        headerStyle: HeaderStyle(
          formatButtonVisible: false,
          titleCentered: true,
          titleTextStyle: TextStyle(
            fontSize: 18,
            fontWeight: FontWeight.bold,
          ),
        ),
        onDaySelected: (selectedDay, focusedDay) {
          controller.onDaySelected(selectedDay, focusedDay);
        },
        onPageChanged: (focusedDay) {
          controller.focusedDate.value = focusedDay;
        },
        calendarBuilders: CalendarBuilders(
          markerBuilder: (context, date, events) {
            if (events.isNotEmpty) {
              return Positioned(
                bottom: 1,
                child: Row(
                  mainAxisAlignment: MainAxisAlignment.center,
                  children: events.take(3).map((event) {
                    return Container(
                      margin: EdgeInsets.symmetric(horizontal: 1),
                      width: 6,
                      height: 6,
                      decoration: BoxDecoration(
                        color: _getEventColor(event.status),
                        shape: BoxShape.circle,
                      ),
                    );
                  }).toList(),
                ),
              );
            }
            return SizedBox();
          },
        ),
      ),
    ));
  }

  Color _getEventColor(ConsultationStatus status) {
    switch (status) {
      case ConsultationStatus.pending:
        return Colors.orange;
      case ConsultationStatus.confirmed:
        return Colors.green;
      case ConsultationStatus.cancelled:
        return Colors.red;
      case ConsultationStatus.completed:
        return Colors.blue;
      case ConsultationStatus.noShow:
        return Colors.grey;
    }
  }
}
```

---

## 🔧 Backend Implementation

### 1. Project Structure
```
backend/
├── cmd/
│   └── main.go
├── internal/
│   ├── api/
│   │   ├── handlers/
│   │   │   ├── auth_handler.go
│   │   │   ├── consultation_handler.go
│   │   │   └── notification_handler.go
│   │   ├── middleware/
│   │   │   ├── auth_middleware.go
│   │   │   └── cors_middleware.go
│   │   └── routes/
│   │       └── routes.go
│   ├── models/
│   │   ├── user.go
│   │   ├── consultation.go
│   │   └── notification.go
│   ├── services/
│   │   ├── consultation_service.go
│   │   ├── notification_service.go
│   │   └── email_service.go
│   ├── repositories/
│   │   ├── consultation_repository.go
│   │   └── user_repository.go
│   └── config/
│       └── database.go
├── pkg/
│   ├── utils/
│   │   ├── jwt.go
│   │   └── validator.go
│   └── scheduler/
│       └── reminder_scheduler.go
└── go.mod
```

### 2. Core Models

#### Consultation Model (Go)
```go
package models

import (
    "time"
    "go.mongodb.org/mongo-driver/bson/primitive"
)

type Consultation struct {
    ID          primitive.ObjectID `json:"id" bson:"_id,omitempty"`
    StudentID   primitive.ObjectID `json:"studentId" bson:"studentId"`
    TutorID     primitive.ObjectID `json:"tutorId" bson:"tutorId"`
    Title       string             `json:"title" bson:"title"`
    Description string             `json:"description" bson:"description"`
    Subject     string             `json:"subject" bson:"subject"`
    Status      ConsultationStatus `json:"status" bson:"status"`
    Schedule    Schedule           `json:"schedule" bson:"schedule"`
    Meeting     Meeting            `json:"meeting" bson:"meeting"`
    Documents   []Document         `json:"documents" bson:"documents"`
    Notes       Notes              `json:"notes" bson:"notes"`
    Rating      *Rating            `json:"rating,omitempty" bson:"rating,omitempty"`
    Reminders   []Reminder         `json:"reminders" bson:"reminders"`
    CreatedAt   time.Time          `json:"createdAt" bson:"createdAt"`
    UpdatedAt   time.Time          `json:"updatedAt" bson:"updatedAt"`
    CancelledAt *time.Time         `json:"cancelledAt,omitempty" bson:"cancelledAt,omitempty"`
    CancellationReason string      `json:"cancellationReason,omitempty" bson:"cancellationReason,omitempty"`
}

type ConsultationStatus string

const (
    StatusPending   ConsultationStatus = "pending"
    StatusConfirmed ConsultationStatus = "confirmed"
    StatusCancelled ConsultationStatus = "cancelled"
    StatusCompleted ConsultationStatus = "completed"
    StatusNoShow    ConsultationStatus = "no_show"
)

type Schedule struct {
    StartTime time.Time `json:"startTime" bson:"startTime"`
    EndTime   time.Time `json:"endTime" bson:"endTime"`
    Duration  int       `json:"duration" bson:"duration"` // in minutes
    Timezone  string    `json:"timezone" bson:"timezone"`
}

type Meeting struct {
    Type             string `json:"type" bson:"type"` // chat (Phase 1), video/in_person/phone (future)
    Platform         string `json:"platform" bson:"platform"` // writeen_chat (Phase 1)
    ChatRoomID       string `json:"chatRoomId,omitempty" bson:"chatRoomId,omitempty"`
    SessionURL       string `json:"sessionUrl,omitempty" bson:"sessionUrl,omitempty"`
    ScheduledDuration int   `json:"scheduledDuration,omitempty" bson:"scheduledDuration,omitempty"` // minutes
    Location         string `json:"location,omitempty" bson:"location,omitempty"` // for future use
}

type Document struct {
    Name       string             `json:"name" bson:"name"`
    URL        string             `json:"url" bson:"url"`
    Type       string             `json:"type" bson:"type"`
    UploadedBy primitive.ObjectID `json:"uploadedBy" bson:"uploadedBy"`
    UploadedAt time.Time          `json:"uploadedAt" bson:"uploadedAt"`
}

type Notes struct {
    StudentNotes   string `json:"studentNotes" bson:"studentNotes"`
    TutorNotes     string `json:"tutorNotes" bson:"tutorNotes"`
    SessionSummary string `json:"sessionSummary" bson:"sessionSummary"`
}

type Rating struct {
    StudentRating int    `json:"studentRating" bson:"studentRating"`
    TutorRating   int    `json:"tutorRating" bson:"tutorRating"`
    Feedback      string `json:"feedback" bson:"feedback"`
}

type Reminder struct {
    Type        string    `json:"type" bson:"type"` // email, push, sms
    ScheduledAt time.Time `json:"scheduledAt" bson:"scheduledAt"`
    Sent        bool      `json:"sent" bson:"sent"`
    SentAt      *time.Time `json:"sentAt,omitempty" bson:"sentAt,omitempty"`
}
```

### 3. Consultation Service
```go
package services

import (
    "context"
    "errors"
    "time"
    
    "writeen-backend/internal/models"
    "writeen-backend/internal/repositories"
)

type ConsultationService struct {
    consultationRepo *repositories.ConsultationRepository
    userRepo         *repositories.UserRepository
    notificationService *NotificationService
}

func NewConsultationService(
    consultationRepo *repositories.ConsultationRepository,
    userRepo *repositories.UserRepository,
    notificationService *NotificationService,
) *ConsultationService {
    return &ConsultationService{
        consultationRepo: consultationRepo,
        userRepo:         userRepo,
        notificationService: notificationService,
    }
}

func (s *ConsultationService) CreateConsultation(ctx context.Context, consultation *models.Consultation) (*models.Consultation, error) {
    // Validate consultation data
    if err := s.validateConsultation(consultation); err != nil {
        return nil, err
    }
    
    // Check for conflicts
    conflicts, err := s.consultationRepo.CheckConflicts(ctx, consultation.TutorID, consultation.Schedule.StartTime, consultation.Schedule.EndTime)
    if err != nil {
        return nil, err
    }
    
    if len(conflicts) > 0 {
        return nil, errors.New("time slot is already booked")
    }
    
    // Set default values
    consultation.Status = models.StatusPending
    consultation.CreatedAt = time.Now()
    consultation.UpdatedAt = time.Now()
    
    // Create reminders
    consultation.Reminders = s.createReminders(consultation.Schedule.StartTime)
    
    // Save to database
    result, err := s.consultationRepo.Create(ctx, consultation)
    if err != nil {
        return nil, err
    }
    
    // Send notification to tutor
    go s.notificationService.SendConsultationRequest(ctx, result)
    
    return result, nil
}

func (s *ConsultationService) UpdateConsultationStatus(ctx context.Context, id string, status models.ConsultationStatus, reason string) error {
    consultation, err := s.consultationRepo.GetByID(ctx, id)
    if err != nil {
        return err
    }
    
    // Update status
    consultation.Status = status
    consultation.UpdatedAt = time.Now()
    
    if status == models.StatusCancelled {
        now := time.Now()
        consultation.CancelledAt = &now
        consultation.CancellationReason = reason
    }
    
    // Save to database
    if err := s.consultationRepo.Update(ctx, consultation); err != nil {
        return err
    }
    
    // Send notifications
    go s.notificationService.SendStatusUpdate(ctx, consultation)
    
    // Schedule reminders for confirmed consultations
    if status == models.StatusConfirmed {
        go s.scheduleReminders(ctx, consultation)
    }
    
    return nil
}

func (s *ConsultationService) GetConsultations(ctx context.Context, userID string, filters map[string]interface{}) ([]*models.Consultation, error) {
    return s.consultationRepo.GetByUserID(ctx, userID, filters)
}

func (s *ConsultationService) GetTutorAvailability(ctx context.Context, tutorID string, startDate, endDate time.Time) ([]models.TimeSlot, error) {
    // Get tutor's availability configuration
    availability, err := s.userRepo.GetTutorAvailability(ctx, tutorID)
    if err != nil {
        return nil, err
    }
    
    // Get existing bookings
    bookings, err := s.consultationRepo.GetBookingsByTutor(ctx, tutorID, startDate, endDate)
    if err != nil {
        return nil, err
    }
    
    // Generate available time slots
    return s.generateAvailableSlots(availability, bookings, startDate, endDate), nil
}

func (s *ConsultationService) validateConsultation(consultation *models.Consultation) error {
    if consultation.Title == "" {
        return errors.New("consultation title is required")
    }
    
    if consultation.Description == "" {
        return errors.New("consultation description is required")
    }
    
    if consultation.Schedule.StartTime.IsZero() {
        return errors.New("start time is required")
    }
    
    if consultation.Schedule.EndTime.IsZero() {
        return errors.New("end time is required")
    }
    
    if consultation.Schedule.StartTime.After(consultation.Schedule.EndTime) {
        return errors.New("start time must be before end time")
    }
    
    if consultation.Schedule.StartTime.Before(time.Now()) {
        return errors.New("cannot schedule consultation in the past")
    }
    
    return nil
}

func (s *ConsultationService) createReminders(startTime time.Time) []models.Reminder {
    reminders := []models.Reminder{
        {
            Type:        "email",
            ScheduledAt: startTime.Add(-24 * time.Hour), // 24 hours before
            Sent:        false,
        },
        {
            Type:        "push",
            ScheduledAt: startTime.Add(-1 * time.Hour), // 1 hour before
            Sent:        false,
        },
        {
            Type:        "push",
            ScheduledAt: startTime.Add(-15 * time.Minute), // 15 minutes before
            Sent:        false,
        },
    }
    
    return reminders
}

func (s *ConsultationService) scheduleReminders(ctx context.Context, consultation *models.Consultation) {
    for _, reminder := range consultation.Reminders {
        if !reminder.Sent && reminder.ScheduledAt.After(time.Now()) {
            go s.notificationService.ScheduleReminder(ctx, consultation, reminder)
        }
    }
}
```

### 4. Notification Service
```go
package services

import (
    "context"
    "fmt"
    "time"
    
    "writeen-backend/internal/models"
    "writeen-backend/pkg/scheduler"
)

type NotificationService struct {
    emailService *EmailService
    pushService  *PushService
    scheduler    *scheduler.ReminderScheduler
}

func NewNotificationService(
    emailService *EmailService,
    pushService *PushService,
    scheduler *scheduler.ReminderScheduler,
) *NotificationService {
    return &NotificationService{
        emailService: emailService,
        pushService:  pushService,
        scheduler:    scheduler,
    }
}

func (s *NotificationService) SendConsultationRequest(ctx context.Context, consultation *models.Consultation) error {
    // Get tutor info
    tutor, err := s.userRepo.GetByID(ctx, consultation.TutorID.Hex())
    if err != nil {
        return err
    }
    
    // Get student info
    student, err := s.userRepo.GetByID(ctx, consultation.StudentID.Hex())
    if err != nil {
        return err
    }
    
    // Send email to tutor
    subject := "New Consultation Request"
    body := fmt.Sprintf(`
        Hello %s,
        
        You have received a new consultation request from %s.
        
        Details:
        - Topic: %s
        - Description: %s
        - Scheduled Time: %s
        
        Please log in to your Writeen account to accept or decline this request.
        
        Best regards,
        Writeen Team
    `, tutor.Profile.FirstName, student.Profile.FirstName, consultation.Title, consultation.Description, consultation.Schedule.StartTime.Format("2006-01-02 15:04"))
    
    if err := s.emailService.SendEmail(tutor.Email, subject, body); err != nil {
        return err
    }
    
    // Send push notification
    pushMessage := fmt.Sprintf("New consultation request from %s", student.Profile.FirstName)
    if err := s.pushService.SendPushNotification(tutor.ID.Hex(), "New Request", pushMessage); err != nil {
        return err
    }
    
    return nil
}

func (s *NotificationService) SendStatusUpdate(ctx context.Context, consultation *models.Consultation) error {
    // Get both user infos
    tutor, err := s.userRepo.GetByID(ctx, consultation.TutorID.Hex())
    if err != nil {
        return err
    }
    
    student, err := s.userRepo.GetByID(ctx, consultation.StudentID.Hex())
    if err != nil {
        return err
    }
    
    var subject, body string
    var recipient *models.User
    
    switch consultation.Status {
    case models.StatusConfirmed:
        recipient = student
        subject = "Consultation Confirmed"
        body = fmt.Sprintf(`
            Hello %s,
            
            Your consultation request has been confirmed by %s.
            
            Details:
            - Topic: %s
            - Scheduled Time: %s
            
            Chat session details will be sent separately.
            
            Best regards,
            Writeen Team
        `, student.Profile.FirstName, tutor.Profile.FirstName, consultation.Title, consultation.Schedule.StartTime.Format("2006-01-02 15:04"))
        
    case models.StatusCancelled:
        recipient = student
        subject = "Consultation Cancelled"
        body = fmt.Sprintf(`
            Hello %s,
            
            Your consultation with %s has been cancelled.
            
            Reason: %s
            
            Please feel free to schedule another consultation.
            
            Best regards,
            Writeen Team
        `, student.Profile.FirstName, tutor.Profile.FirstName, consultation.CancellationReason)
    }
    
    // Send email notification
    if err := s.emailService.SendEmail(recipient.Email, subject, body); err != nil {
        return err
    }
    
    // Send push notification
    pushMessage := fmt.Sprintf("Consultation %s", consultation.Status)
    if err := s.pushService.SendPushNotification(recipient.ID.Hex(), subject, pushMessage); err != nil {
        return err
    }
    
    return nil
}

func (s *NotificationService) ScheduleReminder(ctx context.Context, consultation *models.Consultation, reminder models.Reminder) error {
    // Schedule reminder job
    job := &scheduler.ReminderJob{
        ConsultationID: consultation.ID.Hex(),
        Type:           reminder.Type,
        ScheduledAt:    reminder.ScheduledAt,
        Executed:       false,
    }
    
    return s.scheduler.ScheduleReminder(job)
}

func (s *NotificationService) SendReminder(ctx context.Context, consultationID string, reminderType string) error {
    // Get consultation details
    consultation, err := s.consultationRepo.GetByID(ctx, consultationID)
    if err != nil {
        return err
    }
    
    // Get user info
    student, err := s.userRepo.GetByID(ctx, consultation.StudentID.Hex())
    if err != nil {
        return err
    }
    
    tutor, err := s.userRepo.GetByID(ctx, consultation.TutorID.Hex())
    if err != nil {
        return err
    }
    
    // Calculate time until consultation
    timeUntil := time.Until(consultation.Schedule.StartTime)
    var timeString string
    
    switch {
    case timeUntil > 23*time.Hour:
        timeString = "tomorrow"
    case timeUntil > time.Hour:
        timeString = fmt.Sprintf("in %d hours", int(timeUntil.Hours()))
    case timeUntil > time.Minute:
        timeString = fmt.Sprintf("in %d minutes", int(timeUntil.Minutes()))
    default:
        timeString = "now"
    }
    
    subject := "Consultation Reminder"
    
    switch reminderType {
    case "email":
        // Send email reminder
        body := fmt.Sprintf(`
            Hello %s,
            
            This is a reminder that you have a consultation scheduled %s.
            
            Details:
            - Topic: %s
            - Tutor: %s
            - Time: %s
            
            Chat Session: %s
            
            Best regards,
            Writeen Team
        `, student.Profile.FirstName, timeString, consultation.Title, tutor.Profile.FirstName, consultation.Schedule.StartTime.Format("2006-01-02 15:04"), consultation.Meeting.SessionURL)
        
        if err := s.emailService.SendEmail(student.Email, subject, body); err != nil {
            return err
        }
        
    case "push":
        // Send push notification
        message := fmt.Sprintf("Your chat consultation with %s starts %s", tutor.Profile.FirstName, timeString)
        if err := s.pushService.SendPushNotification(student.ID.Hex(), subject, message); err != nil {
            return err
        }
    }
    
    return nil
}
```

### 5. Reminder Scheduler
```go
package scheduler

import (
    "context"
    "log"
    "time"
    
    "github.com/robfig/cron/v3"
    "writeen-backend/internal/services"
)

type ReminderScheduler struct {
    cron *cron.Cron
    notificationService *services.NotificationService
}

type ReminderJob struct {
    ConsultationID string
    Type           string
    ScheduledAt    time.Time
    Executed       bool
}

func NewReminderScheduler(notificationService *services.NotificationService) *ReminderScheduler {
    return &ReminderScheduler{
        cron: cron.New(),
        notificationService: notificationService,
    }
}

func (s *ReminderScheduler) Start() {
    s.cron.Start()
    log.Println("Reminder scheduler started")
}

func (s *ReminderScheduler) Stop() {
    s.cron.Stop()
    log.Println("Reminder scheduler stopped")
}

func (s *ReminderScheduler) ScheduleReminder(job *ReminderJob) error {
    // Calculate duration until reminder should be sent
    duration := time.Until(job.ScheduledAt)
    
    // Don't schedule if the time has already passed
    if duration <= 0 {
        log.Printf("Reminder time has already passed for consultation %s", job.ConsultationID)
        return nil
    }
    
    // Create a timer for this specific reminder
    timer := time.NewTimer(duration)
    
    go func() {
        <-timer.C
        
        // Send the reminder
        ctx := context.Background()
        if err := s.notificationService.SendReminder(ctx, job.ConsultationID, job.Type); err != nil {
            log.Printf("Failed to send reminder for consultation %s: %v", job.ConsultationID, err)
        } else {
            log.Printf("Reminder sent for consultation %s (type: %s)", job.ConsultationID, job.Type)
        }
    }()
    
    log.Printf("Scheduled reminder for consultation %s at %s", job.ConsultationID, job.ScheduledAt.Format("2006-01-02 15:04:05"))
    
    return nil
}

// Alternative: Use cron for recurring checks
func (s *ReminderScheduler) setupCronJobs() {
    // Check for reminders every minute
    s.cron.AddFunc("* * * * *", func() {
        s.checkPendingReminders()
    })
}

func (s *ReminderScheduler) checkPendingReminders() {
    ctx := context.Background()
    
    // Get all consultations with pending reminders
    consultations, err := s.consultationRepo.GetConsultationsWithPendingReminders(ctx)
    if err != nil {
        log.Printf("Failed to get consultations with pending reminders: %v", err)
        return
    }
    
    for _, consultation := range consultations {
        for i, reminder := range consultation.Reminders {
            if !reminder.Sent && time.Now().After(reminder.ScheduledAt) {
                // Send reminder
                if err := s.notificationService.SendReminder(ctx, consultation.ID.Hex(), reminder.Type); err != nil {
                    log.Printf("Failed to send reminder: %v", err)
                    continue
                }
                
                // Mark as sent
                consultation.Reminders[i].Sent = true
                now := time.Now()
                consultation.Reminders[i].SentAt = &now
                
                // Update in database
                if err := s.consultationRepo.Update(ctx, consultation); err != nil {
                    log.Printf("Failed to update reminder status: %v", err)
                }
            }
        }
    }
}
```

---

## 🔔 Sistem Reminder

### 1. Jenis Reminder
- **24 jam sebelum**: Email reminder dengan detail lengkap
- **1 jam sebelum**: Push notification
- **15 menit sebelum**: Push notification urgent
- **Custom reminder**: User dapat set reminder tambahan

### 2. Implementasi Push Notification (Flutter)
```dart
// pubspec.yaml
dependencies:
  cloud_firestore: ^4.13.6  # For file storage only
  web_socket_channel: ^2.4.0  # For WebSocket notifications

// notification_service.dart
class NotificationService {
  static late WebSocketChannel _channel;
  
  static Future<void> initialize() async {
    // Connect to WebSocket notification service
    _channel = WebSocketChannel.connect(
      Uri.parse('ws://localhost:8080/notifications'),
    );
    
    // Listen for notifications
    _channel.stream.listen(_handleNotification);
  }
  
  static void _handleNotification(dynamic message) {
    try {
      final data = jsonDecode(message);
      final title = data['title'] ?? 'Writeen';
      final body = data['message'] ?? 'You have a new notification';
      
      // Show in-app notification
      _showInAppNotification(title, body, data);
      
      // Handle notification tap
      if (data['consultationId'] != null) {
        Get.toNamed('/consultation/${data['consultationId']}');
      }
    } catch (e) {
      print('Error handling notification: $e');
    }
  }
  
  static void _showInAppNotification(String title, String body, Map<String, dynamic> data) {
    // Show toast notification in app
    Get.snackbar(
      title,
      body,
      snackPosition: SnackPosition.TOP,
      backgroundColor: Colors.blue,
      colorText: Colors.white,
      duration: Duration(seconds: 3),
      onTap: (_) {
        if (data['consultationId'] != null) {
          Get.toNamed('/consultation/${data['consultationId']}');
        }
      },
    );
  }
}
```

### 3. Email Service (Golang)
```go
package services

import (
    "crypto/tls"
    "fmt"
    "net/smtp"
    "strings"
)

type EmailService struct {
    smtpHost     string
    smtpPort     string
    smtpUsername string
    smtpPassword string
}

func NewEmailService(host, port, username, password string) *EmailService {
    return &EmailService{
        smtpHost:     host,
        smtpPort:     port,
        smtpUsername: username,
        smtpPassword: password,
    }
}

func (s *EmailService) SendEmail(to, subject, body string) error {
    from := s.smtpUsername
    
    // Setup headers
    headers := make(map[string]string)
    headers["From"] = from
    headers["To"] = to
    headers["Subject"] = subject
    headers["MIME-Version"] = "1.0"
    headers["Content-Type"] = "text/html; charset=\"UTF-8\""
    
    // Setup message
    message := ""
    for k, v := range headers {
        message += fmt.Sprintf("%s: %s\r\n", k, v)
    }
    message += "\r\n" + body
    
    // Setup authentication
    auth := smtp.PlainAuth("", s.smtpUsername, s.smtpPassword, s.smtpHost)
    
    // Setup TLS
    tlsConfig := &tls.Config{
        InsecureSkipVerify: true,
        ServerName:         s.smtpHost,
    }
    
    // Connect to server
    conn, err := tls.Dial("tcp", s.smtpHost+":"+s.smtpPort, tlsConfig)
    if err != nil {
        return err
    }
    defer conn.Close()
    
    // Create SMTP client
    client, err := smtp.NewClient(conn, s.smtpHost)
    if err != nil {
        return err
    }
    defer client.Quit()
    
    // Authenticate
    if err := client.Auth(auth); err != nil {
        return err
    }
    
    // Send email
    if err := client.Mail(from); err != nil {
        return err
    }
    
    if err := client.Rcpt(to); err != nil {
        return err
    }
    
    writer, err := client.Data()
    if err != nil {
        return err
    }
    defer writer.Close()
    
    _, err = writer.Write([]byte(message))
    return err
}

func (s *EmailService) SendReminderEmail(to, studentName, tutorName, consultationTitle, consultationTime, sessionURL string) error {
    subject := "Consultation Reminder - Writeen"
    
    body := fmt.Sprintf(`
        <!DOCTYPE html>
        <html>
        <head>
            <style>
                body { font-family: Arial, sans-serif; line-height: 1.6; color: #333; }
                .container { max-width: 600px; margin: 0 auto; padding: 20px; }
                .header { background-color: #1E3A8A; color: white; padding: 20px; border-radius: 8px 8px 0 0; }
                .content { background-color: #f8f9fa; padding: 30px; border-radius: 0 0 8px 8px; }
                .button { background-color: #3B82F6; color: white; padding: 12px 24px; text-decoration: none; border-radius: 6px; display: inline-block; margin-top: 20px; }
                .details { background-color: white; padding: 20px; border-radius: 6px; margin: 20px 0; }
            </style>
        </head>
        <body>
            <div class="container">
                <div class="header">
                    <h2>🎓 Consultation Reminder</h2>
                </div>
                <div class="content">
                    <h3>Hello %s,</h3>
                    <p>This is a friendly reminder about your upcoming consultation.</p>
                    
                    <div class="details">
                        <h4>📋 Consultation Details:</h4>
                        <p><strong>Topic:</strong> %s</p>
                        <p><strong>Tutor:</strong> %s</p>
                        <p><strong>Time:</strong> %s</p>
                        <p><strong>Chat Session:</strong> <a href="%s">Join Chat Room</a></p>
                    </div>
                    
                    <p>Please make sure to join the chat session on time. If you need to reschedule, please do so at least 2 hours before the scheduled time.</p>
                    
                    <a href="%s" class="button">View Consultation Details</a>
                    
                    <p>Best regards,<br>The Writeen Team</p>
                </div>
            </div>
        </body>
        </html>
    `, studentName, consultationTitle, tutorName, consultationTime, sessionURL, sessionURL)
    
    return s.SendEmail(to, subject, body)
}
```

---

## 🧪 Testing Strategy

### 1. Unit Testing (Flutter)
```dart
// test/controllers/schedule_controller_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:get/get.dart';
import 'package:mockito/mockito.dart';

class MockApiService extends Mock implements ApiService {}

void main() {
  late ScheduleController controller;
  late MockApiService mockApiService;

  setUp(() {
    mockApiService = MockApiService();
    Get.put<ApiService>(mockApiService);
    controller = ScheduleController();
  });

  group('ScheduleController', () {
    test('should load consultations successfully', () async {
      // Arrange
      final mockConsultations = [
        Consultation(
          id: '1',
          title: 'Test Consultation',
          // ... other fields
        )
      ];
      
      when(mockApiService.getConsultations()).thenAnswer(
        (_) async => ApiResponse(data: mockConsultations)
      );

      // Act
      await controller.loadConsultations();

      // Assert
      expect(controller.consultations.length, 1);
      expect(controller.consultations.first.title, 'Test Consultation');
      expect(controller.isLoading.value, false);
    });

    test('should handle consultation creation', () async {
      // Arrange
      final newConsultation = Consultation(
        title: 'New Consultation',
        // ... other fields
      );
      
      when(mockApiService.createConsultation(any)).thenAnswer(
        (_) async => ApiResponse(data: newConsultation)
      );

      // Act
      await controller.createConsultation(newConsultation);

      // Assert
      expect(controller.consultations.length, 1);
      verify(mockApiService.createConsultation(newConsultation)).called(1);
    });

    test('should group consultations by date correctly', () {
      // Arrange
      final consultations = [
        Consultation(
          id: '1',
          schedule: Schedule(
            startTime: DateTime(2024, 1, 15, 10, 0),
            endTime: DateTime(2024, 1, 15, 11, 0),
          ),
        ),
        Consultation(
          id: '2',
          schedule: Schedule(
            startTime: DateTime(2024, 1, 15, 14, 0),
            endTime: DateTime(2024, 1, 15, 15, 0),
          ),
        ),
      ];
      
      controller.consultations.addAll(consultations);

      // Act
      controller._groupConsultationsByDate();

      // Assert
      final date = DateTime(2024, 1, 15);
      expect(controller.events[date]?.length, 2);
    });
  });
}
```

### 2. Widget Testing (Flutter)
```dart
// test/widgets/calendar_widget_test.dart
import 'package:flutter/material.dart';
import 'package:flutter_test/flutter_test.dart';
import 'package:get/get.dart';
import 'package:table_calendar/table_calendar.dart';

void main() {
  group('CalendarWidget', () {
    testWidgets('should display calendar with events', (WidgetTester tester) async {
      // Arrange
      final controller = ScheduleController();
      Get.put(controller);
      
      controller.consultations.add(Consultation(
        id: '1',
        title: 'Test Consultation',
        schedule: Schedule(
          startTime: DateTime.now(),
          endTime: DateTime.now().add(Duration(hours: 1)),
        ),
      ));

      // Act
      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: CalendarWidget(),
          ),
        ),
      );

      // Assert
      expect(find.byType(TableCalendar), findsOneWidget);
      expect(find.text('Test Consultation'), findsOneWidget);
    });

    testWidgets('should handle day selection', (WidgetTester tester) async {
      // Arrange
      final controller = ScheduleController();
      Get.put(controller);

      await tester.pumpWidget(
        MaterialApp(
          home: Scaffold(
            body: CalendarWidget(),
          ),
        ),
      );

      // Act
      await tester.tap(find.text('15'));
      await tester.pump();

      // Assert
      expect(controller.selectedDate.value.day, 15);
    });
  });
}
```

### 3. Integration Testing (Golang)
```go
// test/integration/consultation_test.go
package integration

import (
    "bytes"
    "encoding/json"
    "net/http"
    "net/http/httptest"
    "testing"
    "time"

    "github.com/stretchr/testify/assert"
    "writeen-backend/internal/models"
)

func TestConsultationEndpoints(t *testing.T) {
    // Setup test server
    router := setupTestRouter()
    
    t.Run("Create Consultation", func(t *testing.T) {
        consultation := models.Consultation{
            Title:       "Test Consultation",
            Description: "Test Description",
            Subject:     "Computer Science",
            Schedule: models.Schedule{
                StartTime: time.Now().Add(24 * time.Hour),
                EndTime:   time.Now().Add(25 * time.Hour),
                Duration:  60,
                Timezone:  "Asia/Jakarta",
            },
        }
        
        jsonBody, _ := json.Marshal(consultation)
        
        req, _ := http.NewRequest("POST", "/api/consultations", bytes.NewBuffer(jsonBody))
        req.Header.Set("Content-Type", "application/json")
        req.Header.Set("Authorization", "Bearer "+testToken)
        
        w := httptest.NewRecorder()
        router.ServeHTTP(w, req)
        
        assert.Equal(t, http.StatusCreated, w.Code)
        
        var response map[string]interface{}
        json.Unmarshal(w.Body.Bytes(), &response)
        
        assert.True(t, response["success"].(bool))
        assert.NotNil(t, response["data"])
    })
    
    t.Run("Get Consultations", func(t *testing.T) {
        req, _ := http.NewRequest("GET", "/api/consultations", nil)
        req.Header.Set("Authorization", "Bearer "+testToken)
        
        w := httptest.NewRecorder()
        router.ServeHTTP(w, req)
        
        assert.Equal(t, http.StatusOK, w.Code)
        
        var response map[string]interface{}
        json.Unmarshal(w.Body.Bytes(), &response)
        
        assert.True(t, response["success"].(bool))
        assert.NotNil(t, response["data"].(map[string]interface{})["consultations"])
    })
    
    t.Run("Update Consultation Status", func(t *testing.T) {
        statusUpdate := map[string]interface{}{
            "status": "confirmed",
            "notes":  "Looking forward to the session",
        }
        
        jsonBody, _ := json.Marshal(statusUpdate)
        
        req, _ := http.NewRequest("PUT", "/api/consultations/"+testConsultationID+"/status", bytes.NewBuffer(jsonBody))
        req.Header.Set("Content-Type", "application/json")
        req.Header.Set("Authorization", "Bearer "+testToken)
        
        w := httptest.NewRecorder()
        router.ServeHTTP(w, req)
        
        assert.Equal(t, http.StatusOK, w.Code)
    })
}
```

### 4. Load Testing
```bash
# install k6
curl https://github.com/grafana/k6/releases/download/v0.47.0/k6-v0.47.0-linux-amd64.tar.gz -L | tar xvz --strip-components 1

# load_test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '1m', target: 10 },
    { duration: '2m', target: 50 },
    { duration: '1m', target: 100 },
    { duration: '2m', target: 100 },
    { duration: '1m', target: 0 },
  ],
};

export default function() {
  let response = http.get('http://localhost:8080/api/consultations', {
    headers: {
      'Authorization': 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...'
    }
  });
  
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  
  sleep(1);
}

# run load test
k6 run load_test.js
```

---

## 🚀 Deployment

### 1. Docker Configuration

#### Dockerfile (Backend)
```dockerfile
# Build stage
FROM golang:1.21-alpine AS builder

WORKDIR /app

# Install git for go modules
RUN apk add --no-cache git

# Copy go mod files
COPY go.mod go.sum ./
RUN go mod download

# Copy source code
COPY . .

# Build the application
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main cmd/main.go

# Final stage
FROM alpine:latest

RUN apk --no-cache add ca-certificates
WORKDIR /root/

# Copy the binary from builder stage
COPY --from=builder /app/main .

# Expose port
EXPOSE 8080

# Run the application
CMD ["./main"]
```

#### docker-compose.yml
```yaml
version: '3.8'

services:
  mongodb:
    image: mongo:7.0
    container_name: writeen-mongodb
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password123
      MONGO_INITDB_DATABASE: writeen
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db
      - ./init-mongo.js:/docker-entrypoint-initdb.d/init-mongo.js:ro

  redis:
    image: redis:7.2-alpine
    container_name: writeen-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  backend:
    build: .
    container_name: writeen-backend
    restart: unless-stopped
    depends_on:
      - mongodb
      - redis
    environment:
      - MONGODB_URI=mongodb://admin:password123@mongodb:27017/writeen?authSource=admin
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=your-super-secret-jwt-key
      - SMTP_HOST=smtp.gmail.com
      - SMTP_PORT=587
      - SMTP_USERNAME=your-email@gmail.com
      - SMTP_PASSWORD=your-app-password
    ports:
      - "8080:8080"
    volumes:
      - ./uploads:/app/uploads

  nginx:
    image: nginx:alpine
    container_name: writeen-nginx
    restart: unless-stopped
    depends_on:
      - backend
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl

volumes:
  mongodb_data:
  redis_data:
```

### 2. CI/CD Pipeline (GitHub Actions)
```yaml
# .github/workflows/deploy.yml
name: Deploy Writeen Backend

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      mongodb:
        image: mongo:7.0
        ports:
          - 27017:27017
        env:
          MONGO_INITDB_ROOT_USERNAME: admin
          MONGO_INITDB_ROOT_PASSWORD: password123

    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Go
      uses: actions/setup-go@v3
      with:
        go-version: 1.21

    - name: Install dependencies
      run: go mod download

    - name: Run tests
      run: go test -v ./...
      env:
        MONGODB_URI: mongodb://admin:password123@localhost:27017/writeen_test?authSource=admin

    - name: Run linter
      uses: golangci/golangci-lint-action@v3
      with:
        version: latest

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
    
    - name: Build and push Docker image
      uses: docker/build-push-action@v4
      with:
        context: .
        push: true
        tags: |
          your-dockerhub-username/writeen-backend:latest
          your-dockerhub-username/writeen-backend:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - name: Deploy to production
      uses: appleboy/ssh-action@v0.1.7
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.SSH_KEY }}
        script: |
          cd /opt/writeen
          docker-compose pull
          docker-compose up -d
          docker system prune -f
```

### 3. Production Configuration

#### Environment Variables
```bash
# .env.production
MONGODB_URI=mongodb://admin:secure_password@localhost:27017/writeen?authSource=admin
REDIS_URL=redis://localhost:6379
JWT_SECRET=your-super-secure-jwt-secret-key-here
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=noreply@writeen.com
SMTP_PASSWORD=your-app-specific-password
FIRESTORE_PROJECT_ID=writeen-app
FIRESTORE_PRIVATE_KEY_ID=your-private-key-id
FIRESTORE_PRIVATE_KEY=your-private-key
FIRESTORE_CLIENT_EMAIL=firebase-adminsdk@writeen-app.iam.gserviceaccount.com
FIRESTORE_CLIENT_ID=your-client-id
APP_ENV=production
APP_PORT=8080
```

#### Nginx Configuration
```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    upstream backend {
        server backend:8080;
    }

    server {
        listen 80;
        server_name api.writeen.com;

        # Redirect HTTP to HTTPS
        return 301 https://$server_name$request_uri;
    }

    server {
        listen 443 ssl http2;
        server_name api.writeen.com;

        ssl_certificate /etc/nginx/ssl/certificate.pem;
        ssl_certificate_key /etc/nginx/ssl/private.key;

        # Security headers
        add_header X-Frame-Options "SAMEORIGIN" always;
        add_header X-Content-Type-Options "nosniff" always;
        add_header X-XSS-Protection "1; mode=block" always;
        add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

        # API routes
        location /api/ {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # CORS headers
            add_header Access-Control-Allow-Origin "*" always;
            add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS" always;
            add_header Access-Control-Allow-Headers "DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization" always;
            
            if ($request_method = 'OPTIONS') {
                add_header Access-Control-Allow-Origin "*";
                add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
                add_header Access-Control-Allow-Headers "DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,Authorization";
                add_header Content-Length 0;
                add_header Content-Type text/plain;
                return 204;
            }
        }

        # Health check
        location /health {
            proxy_pass http://backend;
            access_log off;
        }
    }
}
```

### 4. Monitoring dan Logging

#### Prometheus Configuration
```yaml
# prometheus.yml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'writeen-backend'
    static_configs:
      - targets: ['localhost:8080']
    metrics_path: '/metrics'
    scrape_interval: 10s
```

#### Grafana Dashboard
```json
{
  "dashboard": {
    "title": "Writeen Backend Metrics",
    "panels": [
      {
        "title": "API Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_request_duration_seconds_sum[5m]) / rate(http_request_duration_seconds_count[5m])",
            "legendFormat": "Average Response Time"
          }
        ]
      },
      {
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "Requests per Second"
          }
        ]
      },
      {
        "title": "Error Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total{status=~\"5..\"}[5m])",
            "legendFormat": "5xx Errors"
          }
        ]
      }
    ]
  }
}
```

---

## 📋 Implementation Checklist

### Phase 1: Basic Setup (Week 1)
- [ ] Setup Flutter project structure
- [ ] Setup Golang backend project
- [ ] Configure MongoDB connection
- [ ] Implement basic authentication
- [ ] Create basic UI screens

### Phase 2: Core Features (Week 2)
- [ ] Implement consultation CRUD operations
- [ ] Create calendar integration
- [ ] Build availability management
- [ ] Add conflict checking
- [ ] Implement status updates

### Phase 3: Notifications (Week 3)
- [ ] Setup WebSocket notification service
- [ ] Implement email service
- [ ] Create reminder scheduler
- [ ] Build notification system
- [ ] Add push notification handling

### Phase 4: Polish & Deploy (Week 4)
- [ ] Write comprehensive tests
- [ ] Setup CI/CD pipeline
- [ ] Configure production environment
- [ ] Add monitoring and logging
- [ ] Deploy to production

---

## 🔧 Development Tools

### Frontend (Flutter)
```yaml
# pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  get: ^4.6.6
  dio: ^5.3.2
  table_calendar: ^3.0.9
  cloud_firestore: ^4.13.6  # For file storage only
  web_socket_channel: ^2.4.0  # For WebSocket notifications
  shared_preferences: ^2.2.2
  intl: ^0.18.1

dev_dependencies:
  flutter_test:
    sdk: flutter
  mockito: ^5.4.3
  build_runner: ^2.4.7
  json_annotation: ^4.8.1
  json_serializable: ^6.7.1
```

### Backend (Golang)
```go
// go.mod
module writeen-backend

go 1.21

require (
    github.com/gin-gonic/gin v1.9.1
    github.com/golang-jwt/jwt/v5 v5.0.0
    github.com/robfig/cron/v3 v3.0.1
    go.mongodb.org/mongo-driver v1.13.1
    golang.org/x/crypto v0.15.0
    gopkg.in/gomail.v2 v2.0.0-20160411212932-81ebce5c23df
)
```

---

## 📖 Kesimpulan

Sistem Jadwal Konsultasi Writeen telah dirancang dengan arsitektur yang scalable dan maintainable. Dengan menggunakan Flutter untuk frontend dan Golang untuk backend, sistem ini dapat menangani kebutuhan scheduling yang kompleks dengan fitur-fitur:

1. **Manajemen Jadwal Komprehensif**: CRUD operations lengkap dengan validasi konflik
2. **Integrasi Kalender**: UI yang intuitif dengan multiple view modes
3. **Sistem Reminder Otomatis**: Multiple channel notifications dengan timing yang fleksibel
4. **Real-time Updates**: Push notifications dan email alerts
5. **Scalable Architecture**: Microservices-ready dengan proper separation of concerns

Implementation roadmap selama 4 minggu memungkinkan development yang terstruktur dengan testing dan deployment yang proper. Sistem ini siap untuk dikembangkan lebih lanjut dengan fitur-fitur advanced seperti video calling integration, advanced analytics, dan machine learning recommendations.

**Phase 1 Limitation**: Pada fase ini, konsultasi dilakukan melalui chat room internal. Video calling akan diimplementasikan pada fase pengembangan selanjutnya setelah feature chat sistem selesai dikembangkan.

**Next Steps**: Implementasi Feature 2 (Fitur Chat) dengan integrasi real-time messaging dan file sharing capabilities yang akan menjadi fondasi untuk chat-based consultations. 