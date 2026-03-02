import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';
import 'package:google_fonts/google_fonts.dart';
import 'dart:async';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      theme: ThemeData.dark(),
      home: const LoginPage(),
    );
  }
}

class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() => _LoginPageState();
}

class _LoginPageState extends State<LoginPage> {
  final passwordController = TextEditingController();

  void login() {
    if (passwordController.text == "12345") {
      Navigator.pushReplacement(
          context, MaterialPageRoute(builder: (_) => const Dashboard()));
    } else {
      ScaffoldMessenger.of(context)
          .showSnackBar(const SnackBar(content: Text("Wrong Password")));
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.black,
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(30),
          child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
              Text("NxT OFFICELE Admin",
                  style: GoogleFonts.poppins(
                      fontSize: 22,
                      color: Colors.amber,
                      fontWeight: FontWeight.bold)),
              const SizedBox(height: 20),
              TextField(
                controller: passwordController,
                obscureText: true,
                decoration: const InputDecoration(
                    hintText: "Enter Password",
                    border: OutlineInputBorder()),
              ),
              const SizedBox(height: 15),
              ElevatedButton(
                  style: ElevatedButton.styleFrom(
                      backgroundColor: Colors.amber),
                  onPressed: login,
                  child: const Text("Login",
                      style: TextStyle(color: Colors.black)))
            ],
          ),
        ),
      ),
    );
  }
}

class Dashboard extends StatefulWidget {
  const Dashboard({super.key});

  @override
  State<Dashboard> createState() => _DashboardState();
}

class _DashboardState extends State<Dashboard> {
  List orders = [];
  List filteredOrders = [];
  int totalSales = 0;

  Future fetchOrders() async {
    final response = await http.get(
        Uri.parse("https://yourdomain.com/admin_api.php"));

    if (response.statusCode == 200) {
      final data = json.decode(response.body);
      setState(() {
        orders = data;
        filteredOrders = data;
        totalSales = data.fold(
            0, (sum, item) => sum + int.parse(item['total']));
      });
    }
  }

  void search(String value) {
    setState(() {
      filteredOrders = orders
          .where((o) =>
              o['name'].toLowerCase().contains(value.toLowerCase()) ||
              o['phone'].contains(value))
          .toList();
    });
  }

  @override
  void initState() {
    super.initState();
    fetchOrders();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("Order Dashboard"),
        backgroundColor: Colors.black,
        foregroundColor: Colors.amber,
      ),
      body: RefreshIndicator(
        onRefresh: fetchOrders,
        child: Column(
          children: [
            Padding(
              padding: const EdgeInsets.all(10),
              child: TextField(
                onChanged: search,
                decoration: const InputDecoration(
                    hintText: "Search by Name or Phone",
                    border: OutlineInputBorder()),
              ),
            ),
            Text("Total Sales: ₹$totalSales",
                style: const TextStyle(
                    color: Colors.amber,
                    fontSize: 18,
                    fontWeight: FontWeight.bold)),
            Expanded(
              child: ListView.builder(
                itemCount: filteredOrders.length,
                itemBuilder: (context, index) {
                  final order = filteredOrders[index];
                  return Card(
                    color: Colors.black,
                    child: ListTile(
                      title: Text(order['name'],
                          style: const TextStyle(color: Colors.white)),
                      subtitle: Text(
                          "₹${order['total']} | ${order['phone']}",
                          style:
                              const TextStyle(color: Colors.grey)),
                      trailing: const Icon(Icons.receipt,
                          color: Colors.amber),
                    ),
                  );
                },
              ),
            ),
          ],
        ),
      ),
    );
  }
}
