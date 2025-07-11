# Writeen - Fitur Chat
## Dokumentasi Lengkap Feature 2

### 📋 Daftar Isi
1. [Overview Sistem](#overview-sistem)
2. [Arsitektur Real-time](#arsitektur-real-time)
3. [User Flow](#user-flow)
4. [Database Schema](#database-schema)
5. [API Specification](#api-specification)
6. [Frontend Implementation](#frontend-implementation)
7. [Backend Implementation](#backend-implementation)
8. [WebSocket Integration](#websocket-integration)
9. [File Sharing System](#file-sharing-system)
10. [Testing Strategy](#testing-strategy)
11. [Deployment](#deployment)

---

## 🎯 Overview Sistem

### Tujuan
Membangun sistem chat real-time yang memungkinkan komunikasi efektif antara mahasiswa dan tutor dengan fitur:
- Real-time messaging dengan WebSocket
- File sharing untuk dokumen akademik
- Chat history yang tersimpan permanen
- Integration dengan sistem konsultasi
- Push notifications untuk pesan

### Target Users
- **Mahasiswa**: Berkomunikasi dengan tutor, berbagi dokumen, mengakses chat history
- **Tutor**: Membalas pesan student, memberikan feedback, sharing materi

### Key Features
1. **Real-time Messaging**
   - Instant message delivery
   - Typing indicators
   - Message read receipts
   - Online/offline status
   
2. **File Sharing System**
   - Upload/download documents (.pdf, .docx, .pptx)
   - Image sharing dengan preview
   - File security dan virus scanning
   - Progress tracking untuk upload/download
   
3. **Chat History & Management**
   - Persistent message storage
   - Search dalam chat history
   - Message threading untuk konsultasi
   - Archive conversations

4. **Integration Features**
   - Chat rooms untuk scheduled consultations
   - Auto-create chat saat konsultasi confirmed
   - Link messages ke specific consultations
   - Academic context preservation

---

## 🏗️ Arsitektur Real-time

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    📱 FLUTTER FRONTEND                          │
├─────────────────────────────────────────────────────────────────┤
│  Chat Interface     │  File Manager      │  Notification       │
│  - Message List     │  - Upload Progress │  - Push Messages    │
│  - Input Field      │  - File Preview    │  - Typing Indicators│
│  - Emoji/Stickers   │  - Download Queue  │  - Online Status    │
└─────────────────────────────────────────────────────────────────┘
                              │
                    WebSocket + HTTP/REST
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    🔧 GOLANG BACKEND                            │
├─────────────────────────────────────────────────────────────────┤
│  WebSocket Hub     │  Message Service   │  File Service       │
│  - Connection Mgmt │  - CRUD Operations │  - Upload Handler   │
│  - Room Management │  - Real-time Sync  │  - Security Scan    │
│  - Broadcast Logic │  - History Storage │  - CDN Integration  │
└─────────────────────────────────────────────────────────────────┘
                              │
                         MongoDB + Redis + S3
                              │
┌─────────────────────────────────────────────────────────────────┐
│                    🗄️ DATABASE & STORAGE                        │
├─────────────────────────────────────────────────────────────────┤
│  MongoDB:           │  Redis:            │  S3/MinIO:          │
│  - messages         │  - active_users    │  - uploaded_files   │
│  - conversations    │  - typing_status   │  - images           │
│  - participants     │  - temp_sessions   │  - documents        │
└─────────────────────────────────────────────────────────────────┘
```

### Technology Stack
- **Frontend**: Flutter + Dart + GetX + WebSocket
- **Backend**: Golang + Gin + Gorilla WebSocket
- **Database**: MongoDB (persistent) + Redis (real-time state)
- **File Storage**: AWS S3 / MinIO
- **CDN**: CloudFlare / AWS CloudFront
- **Search**: MongoDB Text Search / Elasticsearch
- **Security**: JWT + File scanning + Encryption

---

## 👥 User Flow

### 1. Chat Creation Flow

```
🎓 STUDENT/TUTOR CHAT INITIATION
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Access Chat   │───▶│  Select Contact │───▶│  Create/Join    │
│   Section       │    │  or Room        │    │  Conversation   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Share Files    │◀───│  Send Messages  │◀───│  Real-time Chat │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  View History   │───▶│  Search Messages│───▶│  Archive Chat   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 2. Consultation Chat Flow

```
📅 CONSULTATION-BASED CHAT
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Consultation   │───▶│  Auto-create    │───▶│  Send Invite    │
│  Confirmed      │    │  Chat Room      │    │  Notification   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Archive Session│◀───│  Consultation   │◀───│  Join Chat Room │
│  & Summary      │    │  Chat Session   │    │  (Both Parties) │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### 3. File Sharing Flow

```
📁 FILE SHARING PROCESS
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Select File    │───▶│  Security Scan  │───▶│  Upload Progress│
│  (Doc/Image)    │    │  & Validation   │    │  & Preview      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                                       │
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Download File  │◀───│  Receive        │◀───│  Send to Chat   │
│  by Recipient   │    │  Notification   │    │  Room           │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## 📊 Database Schema

### 1. Conversations Collection
```json
{
  "_id": "ObjectId",
  "type": "direct|consultation|group",
  "consultationId": "ObjectId", // if type = consultation
  "title": "string", // for consultation chats
  "participants": [
    {
      "userId": "ObjectId",
      "role": "student|tutor",
      "joinedAt": "Date",
      "leftAt": "Date", // null if still active
      "permissions": ["read", "write", "upload", "download"]
    }
  ],
  "settings": {
    "allowFileUpload": true,
    "maxFileSize": 10485760, // 10MB in bytes
    "allowedFileTypes": [".pdf", ".docx", ".jpg", ".png"],
    "autoArchive": false,
    "retention": 365 // days
  },
  "metadata": {
    "lastMessageAt": "Date",
    "messageCount": "number",
    "isArchived": false,
    "archivedAt": "Date",
    "createdBy": "ObjectId"
  },
  "createdAt": "Date",
  "updatedAt": "Date"
}
```

### 2. Messages Collection
```json
{
  "_id": "ObjectId",
  "conversationId": "ObjectId",
  "senderId": "ObjectId",
  "type": "text|file|image|system|sticker",
  "content": {
    "text": "string", // for text messages
    "fileInfo": {
      "fileName": "string",
      "fileSize": "number",
      "fileType": "string",
      "downloadUrl": "string",
      "thumbnailUrl": "string", // for images
      "scanStatus": "pending|clean|infected",
      "uploadProgress": "number" // 0-100
    },
    "systemMessage": {
      "action": "user_joined|user_left|consultation_started|file_shared",
      "data": "object"
    }
  },
  "status": "sending|sent|delivered|read|failed",
  "readBy": [
    {
      "userId": "ObjectId",
      "readAt": "Date"
    }
  ],
  "replyTo": "ObjectId", // for message threading
  "reactions": [
    {
      "userId": "ObjectId",
      "emoji": "string",
      "addedAt": "Date"
    }
  ],
  "editHistory": [
    {
      "editedAt": "Date",
      "previousContent": "string",
      "editedBy": "ObjectId"
    }
  ],
  "metadata": {
    "clientMessageId": "string", // for client-side deduplication
    "deviceInfo": "string",
    "ipAddress": "string"
  },
  "createdAt": "Date",
  "updatedAt": "Date",
  "deletedAt": "Date" // soft delete
}
```

### 3. File Uploads Collection
```json
{
  "_id": "ObjectId",
  "messageId": "ObjectId",
  "conversationId": "ObjectId",
  "uploadedBy": "ObjectId",
  "fileName": "string",
  "originalFileName": "string",
  "fileSize": "number",
  "mimeType": "string",
  "fileHash": "string", // for deduplication
  "storage": {
    "provider": "s3|minio|local",
    "bucket": "string",
    "key": "string",
    "region": "string",
    "url": "string",
    "cdnUrl": "string"
  },
  "security": {
    "scanStatus": "pending|scanning|clean|infected|error",
    "scanResult": "object",
    "scannedAt": "Date",
    "isEncrypted": true,
    "encryptionKey": "string"
  },
  "access": {
    "isPublic": false,
    "allowedUsers": ["ObjectId"],
    "downloadCount": "number",
    "lastAccessedAt": "Date"
  },
  "thumbnails": [
    {
      "size": "small|medium|large",
      "url": "string",
      "dimensions": "string" // "150x150"
    }
  ],
  "createdAt": "Date",
  "updatedAt": "Date",
  "deletedAt": "Date"
}
```

### 4. User Presence Collection (Redis)
```json
{
  "userId": "string",
  "status": "online|away|busy|offline",
  "lastSeen": "timestamp",
  "currentConversations": ["conversationId"],
  "typingIn": "conversationId", // null if not typing
  "deviceInfo": {
    "platform": "android|ios|web",
    "version": "string",
    "socketId": "string"
  },
  "location": {
    "country": "string",
    "timezone": "string"
  }
}
```

### 5. Chat Sessions Collection
```json
{
  "_id": "ObjectId",
  "conversationId": "ObjectId",
  "consultationId": "ObjectId",
  "sessionType": "consultation|general",
  "startTime": "Date",
  "endTime": "Date",
  "duration": "number", // minutes
  "participants": [
    {
      "userId": "ObjectId",
      "joinedAt": "Date",
      "leftAt": "Date",
      "messageCount": "number",
      "filesShared": "number"
    }
  ],
  "summary": {
    "totalMessages": "number",
    "filesShared": "number",
    "keyTopics": ["string"],
    "actionItems": ["string"],
    "sessionNotes": "string"
  },
  "analytics": {
    "responseTime": "number", // average response time
    "engagement": "number", // messages per minute
    "satisfaction": "number" // if rated
  },
  "createdAt": "Date",
  "updatedAt": "Date"
}
```

---

## 🔌 API Specification

### Base URL
```
Development: http://localhost:8080
Production: https://api.writeen.com
WebSocket: ws://localhost:8080/ws (dev) | wss://api.writeen.com/ws (prod)
```

### Authentication
```
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

### 1. Conversation Management

#### GET /api/conversations
```json
Query Parameters:
- page: number (default: 1)
- limit: number (default: 20)
- type: string (direct|consultation|group)
- archived: boolean (default: false)

Response:
{
  "success": true,
  "data": {
    "conversations": [
      {
        "id": "conversation_id",
        "type": "consultation",
        "title": "Thesis Discussion - Chapter 3",
        "consultationId": "consultation_id",
        "participants": [
          {
            "userId": "user_id",
            "role": "student",
            "user": {
              "name": "John Doe",
              "avatar": "https://cdn.writeen.com/avatars/john.jpg"
            }
          }
        ],
        "lastMessage": {
          "content": "Thank you for the feedback!",
          "senderId": "user_id",
          "sentAt": "2024-01-15T14:30:00Z",
          "type": "text"
        },
        "unreadCount": 3,
        "isOnline": true,
        "createdAt": "2024-01-15T10:00:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 15,
      "totalPages": 1
    }
  }
}
```

#### POST /api/conversations
```json
Request:
{
  "type": "direct",
  "participantIds": ["user_id_1", "user_id_2"],
  "title": "General Discussion", // optional
  "consultationId": "consultation_id" // for consultation type
}

Response:
{
  "success": true,
  "message": "Conversation created successfully",
  "data": {
    "conversation": {
      "id": "conversation_id",
      "type": "direct",
      "participants": [...],
      "settings": {...},
      "createdAt": "2024-01-15T10:00:00Z"
    }
  }
}
```

### 2. Message Management

#### GET /api/conversations/:id/messages
```json
Query Parameters:
- page: number (default: 1)
- limit: number (default: 50)
- before: timestamp (for pagination)
- search: string (search in message content)

Response:
{
  "success": true,
  "data": {
    "messages": [
      {
        "id": "message_id",
        "senderId": "user_id",
        "sender": {
          "name": "Dr. Jane Smith",
          "avatar": "https://cdn.writeen.com/avatars/jane.jpg"
        },
        "type": "text",
        "content": {
          "text": "Please review the methodology section again."
        },
        "status": "read",
        "readBy": [
          {
            "userId": "user_id",
            "readAt": "2024-01-15T14:35:00Z"
          }
        ],
        "replyTo": null,
        "reactions": [],
        "createdAt": "2024-01-15T14:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 50,
      "hasMore": false
    }
  }
}
```

#### POST /api/conversations/:id/messages
```json
Request:
{
  "type": "text",
  "content": {
    "text": "Thank you for the feedback! I'll revise the methodology."
  },
  "replyTo": "message_id", // optional
  "clientMessageId": "client_generated_id" // for deduplication
}

Response:
{
  "success": true,
  "message": "Message sent successfully",
  "data": {
    "message": {
      "id": "message_id",
      "conversationId": "conversation_id",
      "senderId": "user_id",
      "type": "text",
      "content": {...},
      "status": "sent",
      "createdAt": "2024-01-15T14:30:00Z"
    }
  }
}
```

### 3. File Upload

#### POST /api/conversations/:id/upload
```json
Request: multipart/form-data
- file: binary
- description: string (optional)
- replyTo: string (optional message ID)

Response:
{
  "success": true,
  "message": "File uploaded successfully",
  "data": {
    "message": {
      "id": "message_id",
      "type": "file",
      "content": {
        "fileInfo": {
          "fileName": "thesis_chapter_3.pdf",
          "fileSize": 2048576,
          "fileType": "application/pdf",
          "downloadUrl": "https://cdn.writeen.com/files/abc123.pdf",
          "thumbnailUrl": "https://cdn.writeen.com/thumbnails/abc123.jpg",
          "scanStatus": "clean"
        }
      },
      "createdAt": "2024-01-15T14:30:00Z"
    },
    "upload": {
      "id": "upload_id",
      "status": "completed",
      "progress": 100
    }
  }
}
```

#### GET /api/files/:id/download
```json
Headers:
Authorization: Bearer <JWT_TOKEN>

Response:
- Content-Type: application/octet-stream
- Content-Disposition: attachment; filename="thesis_chapter_3.pdf"
- Content-Length: 2048576
- X-File-Scan-Status: clean

Binary file content
```

### 4. WebSocket Events

#### Connection
```
ws://localhost:8080/ws?token=<JWT_TOKEN>
```

#### Client → Server Events
```json
// Join conversation
{
  "event": "join_conversation",
  "data": {
    "conversationId": "conversation_id"
  }
}

// Send message
{
  "event": "send_message",
  "data": {
    "conversationId": "conversation_id",
    "type": "text",
    "content": {
      "text": "Hello there!"
    },
    "clientMessageId": "client_123"
  }
}

// Typing indicator
{
  "event": "typing",
  "data": {
    "conversationId": "conversation_id",
    "isTyping": true
  }
}

// Mark message as read
{
  "event": "mark_read",
  "data": {
    "messageId": "message_id",
    "conversationId": "conversation_id"
  }
}
```

#### Server → Client Events
```json
// New message received
{
  "event": "message_received",
  "data": {
    "message": {
      "id": "message_id",
      "conversationId": "conversation_id",
      "senderId": "user_id",
      "sender": {...},
      "type": "text",
      "content": {...},
      "createdAt": "2024-01-15T14:30:00Z"
    }
  }
}

// Typing indicator
{
  "event": "user_typing",
  "data": {
    "conversationId": "conversation_id",
    "userId": "user_id",
    "userName": "Dr. Jane Smith",
    "isTyping": true
  }
}

// Message status update
{
  "event": "message_status",
  "data": {
    "messageId": "message_id",
    "status": "delivered|read",
    "readBy": [...]
  }
}

// User presence update
{
  "event": "user_presence",
  "data": {
    "userId": "user_id",
    "status": "online|away|offline",
    "lastSeen": "2024-01-15T14:30:00Z"
  }
}
```

---

## 📁 File Sharing System

### 1. File Upload Implementation

#### Flutter File Upload Controller
```dart
class FileUploadController extends GetxController {
  final ApiService _apiService = Get.find<ApiService>();
  final WebSocketService _wsService = Get.find<WebSocketService>();
  
  final RxMap<String, UploadProgress> uploadProgress = <String, UploadProgress>{}.obs;
  final RxList<String> allowedFileTypes = <String>[
    '.pdf', '.docx', '.pptx', '.txt',
    '.jpg', '.jpeg', '.png', '.gif',
    '.mp4', '.mov', '.avi' // for future video support
  ].obs;
  
  final int maxFileSize = 10 * 1024 * 1024; // 10MB

  Future<void> uploadFile({
    required String conversationId,
    required File file,
    String? description,
    String? replyTo,
  }) async {
    try {
      // Validate file
      if (!_validateFile(file)) {
        throw Exception('Invalid file type or size');
      }
      
      final uploadId = DateTime.now().millisecondsSinceEpoch.toString();
      
      // Initialize progress tracking
      uploadProgress[uploadId] = UploadProgress(
        id: uploadId,
        fileName: file.name,
        fileSize: file.lengthSync(),
        progress: 0,
        status: UploadStatus.uploading,
      );
      
      // Create multipart request
      final formData = FormData.fromMap({
        'file': await MultipartFile.fromFile(
          file.path,
          filename: file.name,
        ),
        'description': description ?? '',
        'replyTo': replyTo ?? '',
      });
      
      // Upload with progress tracking
      final response = await _apiService.uploadFile(
        conversationId: conversationId,
        formData: formData,
        onSendProgress: (sent, total) {
          final progress = (sent / total * 100).round();
          uploadProgress[uploadId] = uploadProgress[uploadId]!.copyWith(
            progress: progress,
          );
        },
      );
      
      // Update progress to completed
      uploadProgress[uploadId] = uploadProgress[uploadId]!.copyWith(
        status: UploadStatus.completed,
        progress: 100,
        downloadUrl: response.data['downloadUrl'],
      );
      
      Get.snackbar('Success', 'File uploaded successfully');
      
      // Remove from progress tracking after delay
      Timer(Duration(seconds: 3), () {
        uploadProgress.remove(uploadId);
      });
      
    } catch (e) {
      // Update progress to failed
      uploadProgress[uploadId] = uploadProgress[uploadId]!.copyWith(
        status: UploadStatus.failed,
        error: e.toString(),
      );
      
      Get.snackbar('Error', 'Failed to upload file: ${e.toString()}');
    }
  }

  bool _validateFile(File file) {
    // Check file size
    if (file.lengthSync() > maxFileSize) {
      Get.snackbar('Error', 'File size must be less than 10MB');
      return false;
    }
    
    // Check file type
    final extension = path.extension(file.path).toLowerCase();
    if (!allowedFileTypes.contains(extension)) {
      Get.snackbar('Error', 'File type not supported');
      return false;
    }
    
    return true;
  }
}
```

### 2. Backend File Security Service
```go
package services

import (
    "crypto/sha256"
    "fmt"
    "io"
    "mime/multipart"
    "path/filepath"
    "strings"
    "time"

    "github.com/aws/aws-sdk-go/aws"
    "github.com/aws/aws-sdk-go/service/s3"
    "writeen-backend/internal/models"
)

type FileService struct {
    s3Client     *s3.S3
    bucketName   string
    cdnUrl       string
    virusScanner *VirusScanner
}

func (fs *FileService) UploadFile(file multipart.File, header *multipart.FileHeader, userID string) (*models.FileUpload, error) {
    // Validate file
    if err := fs.validateFile(header); err != nil {
        return nil, err
    }
    
    // Generate unique filename
    fileName := fs.generateFileName(header.Filename)
    
    // Calculate file hash
    fileHash, err := fs.calculateFileHash(file)
    if err != nil {
        return nil, err
    }
    
    // Scan for viruses
    scanResult, err := fs.virusScanner.ScanFile(file)
    if err != nil {
        return nil, err
    }
    
    if scanResult.IsInfected {
        return nil, fmt.Errorf("file contains malware: %s", scanResult.ThreatName)
    }
    
    // Upload to S3
    uploadResult, err := fs.uploadToS3(file, fileName, header.Header.Get("Content-Type"))
    if err != nil {
        return nil, err
    }
    
    // Create file upload record
    fileUpload := &models.FileUpload{
        FileName:     header.Filename,
        FileSize:     header.Size,
        MimeType:     header.Header.Get("Content-Type"),
        FileHash:     fileHash,
        Storage: models.StorageInfo{
            Provider: "s3",
            Bucket:   fs.bucketName,
            Key:      fileName,
            URL:      uploadResult.Location,
            CDNUrl:   fmt.Sprintf("%s/%s", fs.cdnUrl, fileName),
        },
        Security: models.SecurityInfo{
            ScanStatus:  "clean",
            ScanResult:  scanResult,
            ScannedAt:   time.Now(),
            IsEncrypted: false,
        },
        CreatedAt: time.Now(),
    }
    
    return fileUpload, nil
}

func (fs *FileService) validateFile(header *multipart.FileHeader) error {
    // Check file size
    const maxFileSize = 10 * 1024 * 1024 // 10MB
    if header.Size > maxFileSize {
        return fmt.Errorf("file size exceeds maximum limit of 10MB")
    }
    
    // Check file extension
    allowedExtensions := map[string]bool{
        ".pdf":  true,
        ".docx": true,
        ".pptx": true,
        ".txt":  true,
        ".jpg":  true,
        ".jpeg": true,
        ".png":  true,
        ".gif":  true,
    }
    
    ext := strings.ToLower(filepath.Ext(header.Filename))
    if !allowedExtensions[ext] {
        return fmt.Errorf("file type %s is not allowed", ext)
    }
    
    return nil
}

func (fs *FileService) generateFileName(originalName string) string {
    ext := filepath.Ext(originalName)
    timestamp := time.Now().Unix()
    hash := sha256.Sum256([]byte(fmt.Sprintf("%s_%d", originalName, timestamp)))
    return fmt.Sprintf("%x%s", hash[:16], ext)
}
```

---

## 🧪 Testing Strategy

### 1. WebSocket Testing (Flutter)
```dart
// test/services/websocket_service_test.dart
import 'package:flutter_test/flutter_test.dart';
import 'package:mockito/mockito.dart';

void main() {
  group('WebSocketService', () {
    late WebSocketService service;
    late MockWebSocketChannel mockChannel;

    setUp(() {
      mockChannel = MockWebSocketChannel();
      service = WebSocketService();
    });

    test('should handle incoming message correctly', () async {
      // Arrange
      final messageData = {
        'event': 'message_received',
        'data': {
          'message': {
            'id': 'msg_123',
            'content': {'text': 'Test message'},
            'senderId': 'user_456'
          }
        }
      };

      // Act
      service.handleIncomingMessage(messageData);

      // Assert
      // Verify message was processed correctly
    });

    test('should send typing indicator', () {
      // Arrange
      const conversationId = 'conv_123';
      const isTyping = true;

      // Act
      service.sendTypingIndicator(conversationId, isTyping);

      // Assert
      verify(mockChannel.sink.add(any)).called(1);
    });
  });
}
```

### 2. Load Testing WebSocket
```javascript
// websocket_load_test.js
import ws from 'k6/ws';
import { check } from 'k6';

export let options = {
  stages: [
    { duration: '1m', target: 50 },
    { duration: '3m', target: 100 },
    { duration: '1m', target: 200 },
    { duration: '2m', target: 200 },
    { duration: '1m', target: 0 },
  ],
};

export default function () {
  const url = 'ws://localhost:8080/ws?token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';
  
  const response = ws.connect(url, {}, function (socket) {
    socket.on('open', function open() {
      console.log('Connected');
      
      // Join a conversation
      socket.send(JSON.stringify({
        event: 'join_conversation',
        data: { conversationId: 'test_conversation_1' }
      }));
    });

    socket.on('message', function (message) {
      const data = JSON.parse(message);
      check(data, {
        'message has event': (msg) => msg.event !== undefined,
        'message has data': (msg) => msg.data !== undefined,
      });
    });

    // Send messages every 5 seconds
    socket.setInterval(function timeout() {
      socket.send(JSON.stringify({
        event: 'send_message',
        data: {
          conversationId: 'test_conversation_1',
          type: 'text',
          content: { text: `Test message ${Date.now()}` },
          clientMessageId: `client_${Date.now()}`
        }
      }));
    }, 5000);

    // Keep connection alive for 30 seconds
    socket.setTimeout(function () {
      socket.close();
    }, 30000);
  });

  check(response, { 'status is 101': (r) => r && r.status === 101 });
}
```

---

## 🚀 Deployment

### 1. Docker Configuration with WebSocket Support
```yaml
# docker-compose.yml
version: '3.8'

services:
  redis:
    image: redis:7.2-alpine
    container_name: writeen-redis
    restart: unless-stopped
    command: redis-server --appendonly yes
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  mongodb:
    image: mongo:7.0
    container_name: writeen-mongodb
    restart: unless-stopped
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: ${MONGO_PASSWORD}
      MONGO_INITDB_DATABASE: writeen
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db

  minio:
    image: minio/minio:latest
    container_name: writeen-minio
    restart: unless-stopped
    environment:
      MINIO_ROOT_USER: ${MINIO_ROOT_USER}
      MINIO_ROOT_PASSWORD: ${MINIO_ROOT_PASSWORD}
    ports:
      - "9000:9000"
      - "9001:9001"
    volumes:
      - minio_data:/data
    command: server /data --console-address ":9001"

  backend:
    build: .
    container_name: writeen-backend
    restart: unless-stopped
    depends_on:
      - mongodb
      - redis
      - minio
    environment:
      - MONGODB_URI=mongodb://admin:${MONGO_PASSWORD}@mongodb:27017/writeen?authSource=admin
      - REDIS_URL=redis://redis:6379
      - JWT_SECRET=${JWT_SECRET}
      - MINIO_ENDPOINT=minio:9000
      - MINIO_ACCESS_KEY=${MINIO_ROOT_USER}
      - MINIO_SECRET_KEY=${MINIO_ROOT_PASSWORD}
      - CDN_BASE_URL=${CDN_BASE_URL}
    ports:
      - "8080:8080"
    volumes:
      - ./uploads:/app/uploads

volumes:
  mongodb_data:
  redis_data:
  minio_data:
```

### 2. Nginx Configuration with WebSocket Support
```nginx
# nginx.conf
events {
    worker_connections 1024;
}

http {
    upstream backend {
        server backend:8080;
    }

    # WebSocket connection upgrade
    map $http_upgrade $connection_upgrade {
        default upgrade;
        '' close;
    }

    server {
        listen 443 ssl http2;
        server_name api.writeen.com;

        ssl_certificate /etc/nginx/ssl/certificate.pem;
        ssl_certificate_key /etc/nginx/ssl/private.key;

        # WebSocket endpoint
        location /ws {
            proxy_pass http://backend;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection $connection_upgrade;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # WebSocket specific timeouts
            proxy_read_timeout 86400s;
            proxy_send_timeout 86400s;
            proxy_connect_timeout 86400s;
        }

        # API routes
        location /api/ {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # File upload size limit
            client_max_body_size 10M;
        }
    }
}
```

---

## 📋 Implementation Checklist

### Phase 1: Basic Chat (Week 1)
- [ ] Setup WebSocket infrastructure
- [ ] Implement basic message CRUD
- [ ] Create conversation management
- [ ] Build Flutter chat UI
- [ ] Setup Redis for real-time state

### Phase 2: File Sharing (Week 2)
- [ ] Implement file upload system
- [ ] Setup MinIO/S3 storage
- [ ] Add file security scanning
- [ ] Create file preview components
- [ ] Add download functionality

### Phase 3: Advanced Features (Week 3)
- [ ] Add typing indicators
- [ ] Implement message status tracking
- [ ] Setup presence management
- [ ] Add message reactions
- [ ] Implement message search

### Phase 4: Integration & Polish (Week 4)
- [ ] Integrate with consultation system
- [ ] Add push notifications
- [ ] Implement comprehensive testing
- [ ] Setup monitoring and logging
- [ ] Deploy to production

---

## 📖 Kesimpulan

Sistem Chat Writeen telah dirancang sebagai fondasi komunikasi real-time yang komprehensif dengan fitur-fitur:

1. **Real-time Messaging**: WebSocket-based dengan typing indicators dan presence tracking
2. **File Sharing**: Upload/download aman dengan virus scanning dan thumbnails
3. **Chat History**: Persistent storage dengan search dan pagination
4. **Integration Ready**: Siap terintegrasi dengan sistem konsultasi
5. **Scalable Architecture**: Redis + MongoDB + S3 untuk performa optimal

**Phase 1 Integration**: Chat system ini menjadi backbone untuk consultation sessions, memungkinkan mahasiswa dan tutor berkomunikasi secara real-time dalam dedicated chat rooms.

**Next Steps**: Implementasi Feature 3 (Upload/Download Dokumen Skripsi) yang akan menggunakan file sharing infrastructure dari chat system ini untuk dokumen akademik yang lebih besar dan kompleks. 🚀