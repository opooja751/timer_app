import 'dart:async';
import 'package:flutter/material.dart';
import 'package:flutter_local_notifications/flutter_local_notifications.dart';
import 'package:android_intent_plus/android_intent.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Timer App',
      theme: ThemeData.dark(useMaterial3: true).copyWith(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        textTheme: const TextTheme(
          bodyLarge: TextStyle(fontSize: 18),
          titleLarge: TextStyle(fontWeight: FontWeight.bold, fontSize: 24),
        ),
      ),
      debugShowCheckedModeBanner: false,
      home: TimerScreen(),
    );
  }
}

class TimerScreen extends StatefulWidget {
  @override
  _TimerScreenState createState() => _TimerScreenState();
}

class _TimerScreenState extends State<TimerScreen> with SingleTickerProviderStateMixin {
  final FlutterLocalNotificationsPlugin flutterLocalNotificationsPlugin = FlutterLocalNotificationsPlugin();
  final TextEditingController _controller = TextEditingController();
  Timer? _timer;
  int _start = 0;

  @override
  void initState() {
    super.initState();
    const androidSettings = AndroidInitializationSettings('@mipmap/ic_launcher');
    const initSettings = InitializationSettings(android: androidSettings);
    flutterLocalNotificationsPlugin.initialize(initSettings);
  }

  void startTimer(int seconds) {
    setState(() => _start = seconds);

    _timer = Timer.periodic(Duration(seconds: 1), (timer) {
      if (_start <= 0) {
        timer.cancel();
        showNotification();
        bringAppToForeground();
      } else {
        setState(() => _start--);
      }
    });

    Future.delayed(Duration(seconds: 1), () {
      AndroidIntent intent = AndroidIntent(
        action: 'android.intent.action.MAIN',
        category: 'android.intent.category.HOME',
      );
      intent.launch();
    });
  }

  Future<void> showNotification() async {
    const androidDetails = AndroidNotificationDetails(
      'timer_channel',
      'Timer Notifications',
      importance: Importance.max,
      priority: Priority.high,
    );
    const platformDetails = NotificationDetails(android: androidDetails);
    await flutterLocalNotificationsPlugin.show(
      0,
      '⏰ Timer Done!',
      'Your countdown has ended.',
      platformDetails,
    );
  }

  void bringAppToForeground() {
    const package = 'com.example.timer_app'; // CHANGE THIS TO YOUR PACKAGE NAME
    const component = 'com.example.timer_app.MainActivity';

    AndroidIntent intent = AndroidIntent(
      action: 'android.intent.action.MAIN',
      category: 'android.intent.category.LAUNCHER',
      package: package,
      componentName: component,
      flags: <int>[268435456],
    );
    intent.launch();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Container(
        width: double.infinity,
        height: double.infinity,
        decoration: const BoxDecoration(
          gradient: LinearGradient(
            colors: [Color(0xFF0F2027), Color(0xFF2C5364)],
            begin: Alignment.topLeft,
            end: Alignment.bottomRight,
          ),
        ),
        child: Center(
          child: Container(
            constraints: BoxConstraints(maxWidth: 350),
            padding: const EdgeInsets.all(24),
            decoration: BoxDecoration(
              borderRadius: BorderRadius.circular(25),
              gradient: LinearGradient(
                colors: [Colors.white10, Colors.white.withOpacity(0.05)],
                begin: Alignment.topLeft,
                end: Alignment.bottomRight,
              ),
              border: Border.all(color: Colors.white12),
              boxShadow: [
                BoxShadow(
                  color: Colors.black54,
                  blurRadius: 20,
                  spreadRadius: 2,
                )
              ],
            ),
            child: Column(
              mainAxisSize: MainAxisSize.min,
              children: [
                Text(
                  '⏳ Set Timer',
                  style: TextStyle(fontSize: 30, fontWeight: FontWeight.bold, color: Colors.white),
                ),
                const SizedBox(height: 25),
                TextField(
                  controller: _controller,
                  keyboardType: TextInputType.number,
                  style: const TextStyle(fontSize: 20, color: Colors.white),
                  decoration: InputDecoration(
                    hintText: 'Enter seconds',
                    hintStyle: TextStyle(color: Colors.white54),
                    filled: true,
                    fillColor: Colors.white12,
                    border: OutlineInputBorder(
                      borderRadius: BorderRadius.circular(14),
                      borderSide: BorderSide.none,
                    ),
                    prefixIcon: Icon(Icons.timer, color: Colors.white60),
                  ),
                ),
                const SizedBox(height: 20),
                ElevatedButton.icon(
                  onPressed: () {
                    final seconds = int.tryParse(_controller.text) ?? 0;
                    if (seconds > 0) startTimer(seconds);
                  },
                  icon: Icon(Icons.play_circle, size: 26),
                  label: Text('Start Timer', style: TextStyle(fontSize: 18)),
                  style: ElevatedButton.styleFrom(
                    backgroundColor: Colors.deepPurpleAccent,
                    padding: const EdgeInsets.symmetric(horizontal: 32, vertical: 14),
                    shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(14)),
                    elevation: 8,
                  ),
                ),
                const SizedBox(height: 30),
                AnimatedSwitcher(
                  duration: Duration(milliseconds: 500),
                  child: _start > 0
                      ? Text(
                          '⏰ $_start seconds left',
                          key: ValueKey(_start),
                          style: TextStyle(fontSize: 26, fontWeight: FontWeight.w600, color: Colors.white),
                        )
                      : Text(
                          '🕒 Ready',
                          key: ValueKey('ready'),
                          style: TextStyle(fontSize: 22, fontWeight: FontWeight.w400, color: Colors.white70),
                        ),
                ),
              ],
            ),
          ),
        ),
      ),
    );
  }
}