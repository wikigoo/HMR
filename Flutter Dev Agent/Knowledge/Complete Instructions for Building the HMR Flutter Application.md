

### **Complete Instructions for Building the HMR Flutter Application (Phase 1 & 2\)**

Below are the exact, step-by-step instructions to initialize the HMR mobile application, tailor it for the Iranian market, and set up the architectural foundation to communicate with your Flowise server.

#### **1\. Project Initialization & Dependencies**

* **Explanation:** We will generate a new Flutter project named hmr\_chatbot, navigate into the project directory, and install the required dependencies. http is used for connecting to your Linux VPS (Flowise API), provider for state management, shared\_preferences for local storage, and flutter\_localizations for Right-to-Left (RTL) Persian language support.  
* **Environment:** Windows PowerShell / CMD  
* **Path:** C:\\Projects\\  
* **Code:**

Bash  
flutter create \--org com.hmr hmr\_chatbot  
cd hmr\_chatbot  
flutter pub add http provider shared\_preferences  
flutter pub add flutter\_localizations \--sdk=flutter

#### **2\. Folder Architecture Setup**

* **Explanation:** To maintain a clean and professional codebase, we need to create specific directories inside the lib folder. This separates our user interface (screens/widgets) from our business logic (services/providers).  
* **Environment:** Windows PowerShell / Cursor Terminal  
* **Path:** C:\\Projects\\hmr\_chatbot\\lib\\  
* **Code:**

Bash  
mkdir screens services providers models widgets

#### **3\. Configuring RTL and Localization (main.dart)**

* **Explanation:** We must replace the default Flutter counter application with the HMR foundation. This code configures the app to strictly use Persian (fa\_IR), ensuring all UI elements, text fields, and chat bubbles are naturally aligned Right-to-Left, which is a core requirement for your target market.  
* **Environment:** Cursor IDE  
* **Path:** C:\\Projects\\hmr\_chatbot\\lib\\main.dart  
* **Code:**

Dart  
import 'package:flutter/material.dart';  
import 'package:flutter\_localizations/flutter\_localizations.dart';

void main() {  
  runApp(const HmrApp());  
}

class HmrApp extends StatelessWidget {  
  const HmrApp({super.key});

  @override  
  Widget build(BuildContext context) {  
    return MaterialApp(  
      title: 'HMR Chatbot',  
      debugShowCheckedModeBanner: false,  
      localizationsDelegates: const \[  
        GlobalMaterialLocalizations.delegate,  
        GlobalWidgetsLocalizations.delegate,  
        GlobalCupertinoLocalizations.delegate,  
      \],  
      supportedLocales: const \[  
        Locale('fa', 'IR'), // Persian RTL Locale  
      \],  
      locale: const Locale('fa', 'IR'),  
      theme: ThemeData(  
        primarySwatch: Colors.blue,  
        // TODO: Add 'Vazirmatn' or 'IRANSans' font family later  
      ),  
      home: const Scaffold(  
        body: Center(  
          child: Text('HMR Chatbot Environment Initialized'),  
        ),  
      ),  
    );  
  }  
}

#### **4\. Building the Flowise API Service Foundation**

* **Explanation:** This is the core service that bridges your mobile application to the Flowise agent running on your Linux VPS. It takes the user's string input, packages it into a JSON payload, sends it to the Flowise Prediction API endpoint via HTTP POST, and returns the AI's text response.  
* **Environment:** Cursor IDE  
* **Path:** C:\\Projects\\hmr\_chatbot\\lib\\services\\api\_service.dart  
* **Code:**

Dart  
import 'dart:convert';  
import 'package:http/http.dart' as http;

class ApiService {  
  // TODO: Replace YOUR\_VPS\_IP and YOUR\_CHATFLOW\_ID with your actual Flowise details  
  static const String flowiseEndpoint \= 'http://YOUR\_VPS\_IP:3000/api/v1/prediction/YOUR\_CHATFLOW\_ID';

  Future\<String\> sendMessage(String message) async {  
    try {  
      final response \= await http.post(  
        Uri.parse(flowiseEndpoint),  
        headers: {'Content-Type': 'application/json'},  
        body: jsonEncode({  
          'question': message,  
          // 'overrideConfig': {'sessionId': 'user\_123'} // Required later for memory/chat history  
        }),  
      );

      if (response.statusCode \== 200) {  
        final data \= jsonDecode(response.body);  
        return data\['text'\];   
      } else {  
        return 'ارتباط با سرور هوش مصنوعی برقرار نشد.'; // Translation: Failed to connect to AI server.  
      }  
    } catch (e) {  
      return 'خطا در شبکه. لطفا اتصال اینترنت خود را بررسی کنید.'; // Translation: Network error. Check your internet connection.  
    }  
  }  
}  
