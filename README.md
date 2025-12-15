# دليل تطبيق NeuroAid - Flutter 🧠📱

## 📋 فهرس المحتويات
1. [نظرة عامة](#نظرة-عامة)
2. [معمارية التطبيق](#معمارية-التطبيق)
3. [الميزات الرئيسية](#الميزات-الرئيسية)
4. [تقسيم الشرح بين الطلاب](#تقسيم-الشرح-بين-الطلاب)
5. [أسئلة المناقشة المتوقعة](#أسئلة-المناقشة-المتوقعة)

---

## 🎯 نظرة عامة

**NeuroAid** هو تطبيق صحي ذكي مبني بـ Flutter لمساعدة المرضى في:
- 🧠 تقييم خطر الإصابة بالسكتة الدماغية
- 👨‍⚕️ حجز مواعيد مع الأطباء
- 💬 الدردشة مع مساعد AI طبي
- 📊 تحليل صور الأشعة

### التقنيات المستخدمة:
- **Flutter** 3.9.2 - للواجهات
- **BLoC Pattern** - لإدارة الحالة
- **Dio** - للاتصال بالـ Backend
- **GetIt** - للـ Dependency Injection

---

## 🏗️ معمارية التطبيق

### البنية العامة:
```
lib/
├── main.dart                    # نقطة البداية
├── neuro_aid.dart              # الـ Root Widget
└── src/
    ├── core/                   # الأساسيات المشتركة
    │   ├── bloc/              # State Management
    │   ├── services/          # الاتصال بالـ Backend
    │   ├── models/            # نماذج البيانات
    │   ├── routes/            # التنقل
    │   └── theme/             # الألوان والتصميم
    │
    ├── features/              # الميزات
    │   ├── auth/             # التسجيل والدخول
    │   ├── home/             # الشاشة الرئيسية
    │   ├── doctors/          # الأطباء
    │   ├── appointment/      # الحجوزات
    │   ├── chat_ai/          # الدردشة الذكية
    │   ├── scan/             # فحص الصور
    │   ├── payment/          # الدفع
    │   └── profile/          # الملف الشخصي
    │
    └── shared/               # مكونات مشتركة
        └── widgets/
```

### معمارية BLoC:
```
UI Layer (Screens)
    ↓
BLoC Layer (Business Logic)
    ↓
Service Layer (API Calls)
    ↓
Backend (Flask + AI)
```

---

## ✨ الميزات الرئيسية

### 1️⃣ التسجيل والدخول (Authentication)
**المسؤول: [الطالب 1]**

#### الملفات:
- `lib/src/features/auth/login_screen.dart`
- `lib/src/features/auth/register_screen.dart`
- `lib/src/core/bloc/auth/auth_cubit.dart`
- `lib/src/core/services/auth_service.dart`

#### كيف يعمل؟

**1. تسجيل مستخدم جديد:**
```dart
// في register_screen.dart
ElevatedButton(
  onPressed: () {
    context.read<AuthCubit>().register(
      email: emailController.text,
      password: passwordController.text,
      name: nameController.text,
      phone: phoneController.text,
    );
  },
  child: Text('تسجيل'),
)
```

**2. AuthCubit يعالج الطلب:**
```dart
// في auth_cubit.dart
Future<void> register({
  required String email,
  required String password,
  required String name,
  required String phone,
}) async {
  emit(AuthLoading());
  try {
    final response = await _authService.register(
      email: email,
      password: password,
      name: name,
      phone: phone,
    );
    emit(AuthAuthenticated(response.user, response.accessToken));
  } catch (e) {
    emit(AuthError(e.toString()));
  }
}
```

**3. AuthService يتصل بالـ Backend:**
```dart
// في auth_service.dart
Future<AuthResponse> register({
  required String email,
  required String password,
  required String name,
  required String phone,
}) async {
  final response = await _apiService.post(
    ApiConstants.authRegister, // '/api/main/auth/register'
    data: {
      'email': email,
      'password': password,
      'name': name,
      'phone': phone,
    },
  );
  
  final authResponse = AuthResponse.fromJson(response.data);
  await _saveAuthData(authResponse); // حفظ Token محلياً
  _apiService.setToken(authResponse.accessToken);
  
  return authResponse;
}
```

#### الميزات:
- ✅ تشفير كلمات المرور (bcrypt في Backend)
- ✅ JWT Token للمصادقة
- ✅ حفظ بيانات الدخول محلياً (SharedPreferences)
- ✅ Auto-login عند فتح التطبيق

---

### 2️⃣ الشاشة الرئيسية (Home)
**المسؤول: [الطالب 2]**

#### الملفات:
- `lib/src/features/home/home_screen.dart`
- `lib/src/features/home/HomeTab.dart`
- `lib/src/features/home/DoctorCardsSection.dart`

#### المكونات:

**1. Bottom Navigation:**
```dart
// في CustomBottomNavigationBar.dart
BottomNavigationBar(
  currentIndex: _selectedIndex,
  items: [
    BottomNavigationBarItem(icon: Icon(Icons.home), label: 'الرئيسية'),
    BottomNavigationBarItem(icon: Icon(Icons.calendar), label: 'المواعيد'),
    BottomNavigationBarItem(icon: Icon(Icons.chat), label: 'الدردشة'),
    BottomNavigationBarItem(icon: Icon(Icons.person), label: 'الملف'),
  ],
  onTap: (index) {
    setState(() => _selectedIndex = index);
  },
)
```

**2. عرض الأطباء:**
```dart
// في DoctorCardsSection.dart
BlocBuilder<DoctorsCubit, DoctorsState>(
  builder: (context, state) {
    if (state is DoctorsLoaded) {
      return ListView.builder(
        itemCount: state.doctors.length,
        itemBuilder: (context, index) {
          final doctor = state.doctors[index];
          return DoctorCard(
            doctor: doctor,
            onTap: () => Navigator.pushNamed(
              context,
              AppRouter.doctorInfo,
              arguments: doctor,
            ),
          );
        },
      );
    }
    return CircularProgressIndicator();
  },
)
```

---

### 3️⃣ الأطباء والحجوزات (Doctors & Appointments)
**المسؤول: [الطالب 3]**

#### الملفات:
- `lib/src/features/doctors/doctors_list_screen.dart`
- `lib/src/features/doctors/doctor_info_screen.dart`
- `lib/src/features/appointment/schedule_screen.dart`
- `lib/src/core/services/booking_service.dart`

#### حجز موعد:

**1. اختيار التاريخ:**
```dart
// في schedule_screen.dart
TableCalendar(
  firstDay: DateTime.now(),
  lastDay: DateTime.now().add(Duration(days: 90)),
  selectedDayPredicate: (day) => isSameDay(_selectedDate, day),
  onDaySelected: (selectedDay, focusedDay) {
    setState(() {
      _selectedDate = selectedDay;
      _loadAvailableSlots(); // جلب المواعيد المتاحة
    });
  },
)
```

**2. اختيار الوقت:**
```dart
Wrap(
  children: availableSlots.map((slot) {
    return ChoiceChip(
      label: Text(slot),
      selected: _selectedTime == slot,
      onSelected: (selected) {
        setState(() => _selectedTime = slot);
      },
    );
  }).toList(),
)
```

**3. تأكيد الحجز:**
```dart
ElevatedButton(
  onPressed: () async {
    await context.read<BookingCubit>().createBooking(
      doctorId: widget.doctor.id,
      date: _selectedDate,
      time: _selectedTime,
    );
  },
  child: Text('تأكيد الحجز'),
)
```

**4. BookingService:**
```dart
// في booking_service.dart
Future<Booking> createBooking({
  required int doctorId,
  required DateTime date,
  required String time,
}) async {
  final response = await _apiService.post(
    ApiConstants.bookings,
    data: {
      'doctorId': doctorId,
      'date': date.toIso8601String(),
      'time': time,
      'status': 'pending',
    },
  );
  return Booking.fromJson(response.data);
}
```

---

### 4️⃣ الدردشة الذكية (AI Chat)
**المسؤول: [الطالب 4]**

#### الملفات:
- `lib/src/features/chat_ai/chat_ai_screen.dart`
- `lib/src/core/services/chat_service.dart`

#### كيف يعمل؟

**1. إرسال رسالة:**
```dart
// في chat_ai_screen.dart
void _sendMessage() async {
  final message = _textController.text;
  
  setState(() {
    _messages.add(ChatMessage(
      text: message,
      isUser: true,
      timestamp: DateTime.now(),
    ));
  });
  
  _textController.clear();
  
  // إرسال للـ AI
  final response = await ChatService().sendMessage(
    message: message,
    conversationHistory: _messages,
  );
  
  setState(() {
    _messages.add(ChatMessage(
      text: response,
      isUser: false,
      timestamp: DateTime.now(),
    ));
  });
}
```

**2. ChatService:**
```dart
// في chat_service.dart
Future<String> sendMessage({
  required String message,
  required List<ChatMessage> conversationHistory,
}) async {
  final response = await _apiService.post(
    ApiConstants.aiChat, // '/api/ai/chat'
    data: {
      'message': message,
      'conversationHistory': conversationHistory
          .map((m) => {'role': m.isUser ? 'user' : 'assistant', 'content': m.text})
          .toList(),
    },
  );
  
  return response.data['response'];
}
```

**3. عرض الرسائل:**
```dart
ListView.builder(
  reverse: true,
  itemCount: _messages.length,
  itemBuilder: (context, index) {
    final message = _messages[index];
    return ChatBubble(
      message: message.text,
      isUser: message.isUser,
      timestamp: message.timestamp,
    );
  },
)
```

---

### 5️⃣ فحص الصور (Image Scan)
**المسؤول: [الطالب 5]**

#### الملفات:
- `lib/src/features/scan/scan_screen.dart`
- `lib/src/core/services/scan_service.dart`

#### كيف يعمل؟

**1. اختيار صورة:**
```dart
// في scan_screen.dart
import 'package:image_picker/image_picker.dart';

Future<void> _pickImage() async {
  final picker = ImagePicker();
  final pickedFile = await picker.pickImage(
    source: ImageSource.gallery, // أو ImageSource.camera
  );
  
  if (pickedFile != null) {
    setState(() {
      _selectedImage = File(pickedFile.path);
    });
  }
}
```

**2. رفع الصورة للتحليل:**
```dart
Future<void> _analyzeScan() async {
  if (_selectedImage == null) return;
  
  setState(() => _isAnalyzing = true);
  
  try {
    final result = await ScanService().uploadScan(_selectedImage!);
    
    setState(() {
      _scanResult = result;
      _isAnalyzing = false;
    });
    
    // عرض النتيجة
    _showResultDialog(result);
  } catch (e) {
    setState(() => _isAnalyzing = false);
    _showErrorDialog(e.toString());
  }
}
```

**3. ScanService:**
```dart
// في scan_service.dart
Future<Map<String, dynamic>> uploadScan(File imageFile) async {
  // إنشاء FormData
  FormData formData = FormData.fromMap({
    'image': await MultipartFile.fromFile(
      imageFile.path,
      filename: imageFile.path.split('/').last,
    ),
  });
  
  // إرسال للـ Backend
  final response = await _apiService.post(
    ApiConstants.scans, // '/api/main/scans'
    data: formData,
    options: Options(
      contentType: Headers.multipartFormDataContentType,
    ),
  );
  
  return response.data;
}
```

**4. عرض النتيجة:**
```dart
void _showResultDialog(Map<String, dynamic> result) {
  showDialog(
    context: context,
    builder: (context) => AlertDialog(
      title: Text('نتيجة الفحص'),
      content: Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          Text('النتيجة: ${result['result']}'),
          Text('الدقة: ${(result['confidence'] * 100).toStringAsFixed(1)}%'),
          SizedBox(height: 16),
          ...result['findings'].map<Widget>((f) => Text('• $f')).toList(),
        ],
      ),
      actions: [
        TextButton(
          onPressed: () => Navigator.pop(context),
          child: Text('حسناً'),
        ),
      ],
    ),
  );
}
```

---

### 6️⃣ الدفع (Payment)
**المسؤول: [الطالب 6]**

#### الملفات:
- `lib/src/features/payment/payment_summary_screen.dart`
- `lib/src/features/payment/add_card_screen.dart`
- `lib/src/features/payment/payment_success_screen.dart`

#### كيف يعمل؟

**1. عرض ملخص الدفع:**
```dart
// في payment_summary_screen.dart
Column(
  children: [
    ListTile(
      title: Text('رسوم الاستشارة'),
      trailing: Text('${doctor.consultationFee} جنيه'),
    ),
    ListTile(
      title: Text('ضريبة القيمة المضافة'),
      trailing: Text('${(doctor.consultationFee * 0.14).toStringAsFixed(2)} جنيه'),
    ),
    Divider(),
    ListTile(
      title: Text('الإجمالي', style: TextStyle(fontWeight: FontWeight.bold)),
      trailing: Text(
        '${(doctor.consultationFee * 1.14).toStringAsFixed(2)} جنيه',
        style: TextStyle(fontWeight: FontWeight.bold),
      ),
    ),
  ],
)
```

**2. إضافة بطاقة:**
```dart
// في add_card_screen.dart
import 'package:flutter_credit_card/flutter_credit_card.dart';

CreditCardWidget(
  cardNumber: _cardNumber,
  expiryDate: _expiryDate,
  cardHolderName: _cardHolderName,
  cvvCode: _cvvCode,
  showBackView: _isCvvFocused,
)

CreditCardForm(
  cardNumber: _cardNumber,
  expiryDate: _expiryDate,
  cardHolderName: _cardHolderName,
  cvvCode: _cvvCode,
  onCreditCardModelChange: (CreditCardModel data) {
    setState(() {
      _cardNumber = data.cardNumber;
      _expiryDate = data.expiryDate;
      _cardHolderName = data.cardHolderName;
      _cvvCode = data.cvvCode;
    });
  },
)
```

---

## 🔧 الاتصال بالـ Backend

### API Service Layer:

**1. ApiService (الطبقة الأساسية):**
```dart
// في api_service.dart
class ApiService {
  late final Dio _dio;
  
  ApiService() {
    _dio = Dio(BaseOptions(
      baseUrl: ApiConstants.baseUrl, // 'http://192.168.1.6:8080'
      connectTimeout: Duration(seconds: 30),
      receiveTimeout: Duration(seconds: 30),
    ));
    
    // إضافة Interceptors للـ logging
    _dio.interceptors.add(InterceptorsWrapper(
      onRequest: (options, handler) {
        if (_token != null) {
          options.headers['Authorization'] = 'Bearer $_token';
        }
        return handler.next(options);
      },
    ));
  }
  
  Future<Response> get(String endpoint) async {
    return await _dio.get(endpoint);
  }
  
  Future<Response> post(String endpoint, {dynamic data}) async {
    return await _dio.post(endpoint, data: data);
  }
}
```

**2. API Constants:**
```dart
// في api_constants.dart
class ApiConstants {
  // Gateway URL
  static String get baseUrl {
    if (kDebugMode) {
      return 'http://192.168.1.6:8080'; // للأجهزة الحقيقية
    }
    return 'http://10.0.2.2:8080'; // للـ Emulator
  }
  
  // Auth Endpoints
  static const String authLogin = '/api/main/auth/login';
  static const String authRegister = '/api/main/auth/register';
  
  // Doctors Endpoints
  static const String doctors = '/api/main/doctors';
  static String doctorById(String id) => '/api/main/doctors/$id';
  
  // Bookings Endpoints
  static const String bookings = '/api/main/bookings';
  
  // AI Endpoints
  static const String aiChat = '/api/ai/chat';
  static const String scans = '/api/main/scans';
}
```

---

## 👥 تقسيم الشرح بين الطلاب

### الطالب 1: المقدمة + Authentication (7 دقائق)
**المسؤوليات:**
- شرح معمارية التطبيق العامة
- شرح BLoC Pattern
- شرح Authentication Flow
- عرض Login & Register

**النقاط المهمة:**
- كيف يتم حفظ الـ Token
- كيف يتم التحقق من تسجيل الدخول عند فتح التطبيق
- أمثلة من الكود

---

### الطالب 2: Home & Navigation (5 دقائق)
**المسؤوليات:**
- شرح الشاشة الرئيسية
- شرح Bottom Navigation
- شرح Routing System

**النقاط المهمة:**
- كيف يتم التنقل بين الشاشات
- كيف يتم عرض الأطباء
- أمثلة من الكود

---

### الطالب 3: Doctors & Appointments (7 دقائق)
**المسؤوليات:**
- شرح قائمة الأطباء
- شرح نظام الحجز
- شرح Calendar Integration

**النقاط المهمة:**
- كيف يتم جلب بيانات الأطباء
- كيف يتم حجز موعد
- كيف يتم عرض المواعيد
- أمثلة من الكود

---

### الطالب 4: AI Chat (5 دقائق)
**المسؤوليات:**
- شرح واجهة الدردشة
- شرح الاتصال بـ AI Backend
- عرض Demo

**النقاط المهمة:**
- كيف يتم إرسال الرسائل
- كيف يتم الاحتفاظ بتاريخ المحادثة
- أمثلة من الكود

---

### الطالب 5: Image Scan (5 دقائق)
**المسؤوليات:**
- شرح اختيار الصور
- شرح رفع الصور
- شرح عرض النتائج

**النقاط المهمة:**
- كيف يتم اختيار صورة من الجهاز
- كيف يتم رفعها للـ Backend
- كيف يتم عرض النتيجة
- أمثلة من الكود

---

### الطالب 6: Payment & Profile (5 دقائق)
**المسؤوليات:**
- شرح نظام الدفع
- شرح الملف الشخصي
- الختام والخطط المستقبلية

**النقاط المهمة:**
- كيف يتم إدخال بيانات البطاقة
- كيف يتم عرض الملف الشخصي
- التحديات والحلول

---

## 🎓 أسئلة المناقشة المتوقعة

### أسئلة عامة:

**س1: ليه اخترتوا Flutter؟**
- **الإجابة:** 
  - Cross-platform (Android + iOS بكود واحد)
  - أداء عالي (قريب من Native)
  - UI جميلة ومرنة
  - Hot Reload للتطوير السريع

**س2: إيه الفرق بين StatelessWidget و StatefulWidget؟**
- **الإجابة:**
  - StatelessWidget: لا يتغير (ثابت)
  - StatefulWidget: يمكن أن يتغير (له State)
  - مثال: الزرار ثابت (Stateless)، لكن الـ Counter متغير (Stateful)

**س3: ليه استخدمتوا BLoC بدل Provider أو GetX؟**
- **الإجابة:**
  - BLoC أكثر تنظيماً للمشاريع الكبيرة
  - فصل واضح بين UI و Business Logic
  - سهل في الـ Testing
  - مدعوم رسمياً من Flutter

---

### أسئلة عن Authentication:

**س4: إزاي بتحفظوا الـ Token؟**
- **الإجابة:** باستخدام SharedPreferences - بيحفظ البيانات محلياً على الجهاز

**س5: لو الـ Token انتهت صلاحيته؟**
- **الإجابة:** Backend يرجع error 401، وإحنا بنوجه المستخدم لشاشة Login

**س6: إزاي بتتأكدوا إن المستخدم مسجل دخول؟**
- **الإجابة:** 
```dart
Future<void> checkAuthStatus() async {
  final token = await _authService.getToken();
  if (token != null) {
    emit(AuthAuthenticated(user, token));
  } else {
    emit(AuthUnauthenticated());
  }
}
```

---

### أسئلة عن State Management:

**س7: إيه دور الـ Cubit؟**
- **الإجابة:** 
  - بيدير الـ State (الحالة)
  - بيعالج الـ Business Logic
  - بيتصل بالـ Services
  - بيبلغ الـ UI بالتغييرات

**س8: إيه الفرق بين Cubit و Bloc؟**
- **الإجابة:**
  - Cubit: أبسط، بيستخدم Functions
  - Bloc: أكثر تعقيداً، بيستخدم Events
  - إحنا استخدمنا Cubit عشان أبسط

---

### أسئلة عن API Integration:

**س9: إزاي بتتعاملوا مع الأخطاء من Backend؟**
- **الإجابة:**
```dart
try {
  final response = await _apiService.post(endpoint, data: data);
  return response.data;
} on DioException catch (e) {
  if (e.response?.statusCode == 401) {
    throw 'غير مصرح. سجل دخول مرة أخرى';
  } else if (e.response?.statusCode == 500) {
    throw 'خطأ في الخادم';
  }
  throw 'حدث خطأ غير متوقع';
}
```

**س10: إزاي بتتأكدوا إن الـ API شغال؟**
- **الإجابة:** 
  - بنعمل health check endpoint: `/health`
  - بنستخدم try-catch في كل request
  - بنعرض رسائل خطأ واضحة للمستخدم

---

### أسئلة عن UI/UX:

**س11: إزاي بتعرضوا Loading State؟**
- **الإجابة:**
```dart
BlocBuilder<AuthCubit, AuthState>(
  builder: (context, state) {
    if (state is AuthLoading) {
      return CircularProgressIndicator();
    } else if (state is AuthError) {
      return Text('خطأ: ${state.message}');
    } else if (state is AuthAuthenticated) {
      return HomeScreen();
    }
    return LoginScreen();
  },
)
```

**س12: إزاي بتتعاملوا مع الـ Responsive Design؟**
- **الإجابة:**
  - MediaQuery للحصول على حجم الشاشة
  - LayoutBuilder للتخطيطات المختلفة
  - Flexible & Expanded للمساحات المرنة

---

### أسئلة عن Performance:

**س13: إزاي بتحسنوا أداء التطبيق؟**
- **الإجابة:**
  - Lazy Loading للقوائم الطويلة
  - Caching للصور والبيانات
  - تقليل عدد الـ rebuilds
  - استخدام const constructors

**س14: إزاي بتتعاملوا مع الصور الكبيرة؟**
- **الإجابة:**
  - ضغط الصورة قبل الرفع
  - استخدام Image.network مع caching
  - عرض placeholder أثناء التحميل

---

### أسئلة تقنية متقدمة:

**س15: إزاي بتعملوا Testing؟**
- **الإجابة:**
  - Unit Tests للـ Services
  - Widget Tests للـ UI
  - Integration Tests للـ Flows الكاملة

**س16: إيه الـ Dependencies اللي استخدمتوها؟**
- **الإجابة:**
```yaml
dependencies:
  flutter_bloc: ^8.1.5      # State Management
  dio: ^5.4.0               # HTTP Client
  get_it: ^7.7.0            # Dependency Injection
  shared_preferences: ^2.2.2 # Local Storage
  image_picker: ^1.0.7      # اختيار الصور
  table_calendar: ^3.1.2    # التقويم
  flutter_credit_card: ^4.0.1 # بطاقات الدفع
```

**س17: لو عايزين ندعم أكتر من لغة؟**
- **الإجابة:** نستخدم package زي `flutter_localizations` و `intl`

---

## 📊 مثال على Flow كامل

### مثال: حجز موعد مع دكتور

**1. المستخدم يفتح التطبيق:**
```
SplashScreen → AuthCubit.checkAuthStatus() → HomeScreen
```

**2. يختار دكتور:**
```
HomeScreen → DoctorCard.onTap() → DoctorInfoScreen
```

**3. يضغط "احجز موعد":**
```
DoctorInfoScreen → Navigator.push(ScheduleScreen)
```

**4. يختار تاريخ ووقت:**
```
ScheduleScreen → setState({selectedDate, selectedTime})
```

**5. يؤكد الحجز:**
```
ConfirmButton.onPressed() 
  → BookingCubit.createBooking()
  → BookingService.createBooking()
  → ApiService.post('/api/main/bookings')
  → Backend يحفظ الحجز
  → BookingCubit.emit(BookingSuccess)
  → Navigator.push(PaymentSummaryScreen)
```

**6. يدفع:**
```
PaymentSummaryScreen 
  → Navigator.push(AddCardScreen)
  → يدخل بيانات البطاقة
  → Navigator.push(PaymentSuccessScreen)
```

---

## 🔧 نصائح للعرض

### قبل العرض:
1. ✅ جهزوا Demo شغال على جهاز حقيقي
2. ✅ اتأكدوا إن الـ Backend شغال
3. ✅ جهزوا screenshots لكل شاشة
4. ✅ اقروا الكود كويس
5. ✅ تدربوا على الشرح

### أثناء العرض:
1. 📱 ابدأوا بـ Demo حي
2. 💻 وروا الكود بعدين
3. 🎯 ركزوا على الأجزاء المهمة
4. ⏱️ التزموا بالوقت
5. 😊 كونوا واثقين

### للإجابة على الأسئلة:
1. استمعوا كويس للسؤال
2. لو مش عارفين، قولوا "هنراجع ده ونرد عليك"
3. استخدموا أمثلة من الكود
4. كونوا صادقين

---

## 📚 مصادر للمراجعة

### Documentation:
- [Flutter Docs](https://docs.flutter.dev/)
- [BLoC Library](https://bloclibrary.dev/)
- [Dio Package](https://pub.dev/packages/dio)

### Tutorials:
- Flutter BLoC Pattern
- Flutter State Management
- Flutter API Integration

---

## 🚀 الخطط المستقبلية

### ما تم إنجازه: ✅
- ✅ Authentication كامل
- ✅ عرض الأطباء والحجز
- ✅ AI Chatbot
- ✅ فحص الصور
- ✅ نظام الدفع
- ✅ الملف الشخصي

### قيد التطوير: 🔄
- 🔄 Push Notifications
- 🔄 Dark Mode
- 🔄 Multi-language Support

### المخطط: 📋
- 📋 Video Calls مع الأطباء
- 📋 Health Records
- 📋 Medication Reminders
- 📋 Integration مع Wearables

---

**بالتوفيق في العرض! 🎉**

**فريق Delta - مشروع التخرج 2025**
