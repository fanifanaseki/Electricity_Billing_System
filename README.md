// ========================= pubspec.yaml =========================
name: bdccoin_app
description: BDCCOIN – full authentic app (wallet, community, charts, AI updater, Firebase, Twitter)
publish_to: "none"
version: 1.0.0+1

environment:
  sdk: ">=3.3.0 <4.0.0"

dependencies:
  flutter:
    sdk: flutter
  flutter_riverpod: ^2.5.1
  firebase_core: ^3.3.0
  cloud_firestore: ^5.4.2
  firebase_auth: ^5.1.2
  firebase_remote_config: ^5.0.4
  webview_flutter: ^4.8.0
  qr_flutter: ^4.1.0
  qr_code_scanner: ^1.0.1
  shared_preferences: ^2.2.3
  http: ^1.2.2
  intl: ^0.19.0

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^3.0.2

flutter:
  uses-material-design: true


// ========================= lib/main.dart =========================
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:firebase_core/firebase_core.dart';

import 'firebase_options.dart';
import 'app.dart';
import 'core/ai_updater/auto_update.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  await AiUpdater().checkForUpdates();
  runApp(const ProviderScope(child: BDCCoinApp()));
}


// ========================= lib/app.dart =========================
import 'package:flutter/material.dart';
import 'routes.dart';

class BDCCoinApp extends StatelessWidget {
  const BDCCoinApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'BDCCOIN',
      debugShowCheckedModeBanner: false,
      theme: ThemeData.dark(useMaterial3: true).copyWith(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.tealAccent, brightness: Brightness.dark),
        cardTheme: CardTheme(
          color: Colors.black54,
          shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
        ),
      ),
      home: const HomeRouter(),
    );
  }
}


// ========================= lib/routes.dart =========================
import 'package:flutter/material.dart';

import 'features/wallet/presentation/wallet_screen.dart';
import 'features/transactions/transaction_screen.dart';
import 'features/community/community_feed.dart';
import 'features/charts/tradingview_chart.dart';

class HomeRouter extends StatefulWidget {
  const HomeRouter({super.key});
  @override
  State<HomeRouter> createState() => _HomeRouterState();
}

class _HomeRouterState extends State<HomeRouter> {
  int _index = 0;
  final _screens = const [
    WalletScreen(),
    TransactionScreen(),
    CommunityFeed(),
    TradingViewChart(),
  ];
  final _labels = const ['Wallet', 'Transactions', 'Community', 'Charts'];
  final _icons = const [Icons.account_balance_wallet, Icons.swap_horiz, Icons.group, Icons.show_chart];

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text('BDCCOIN – ${_labels[_index]}'), centerTitle: true),
      body: _screens[_index],
      bottomNavigationBar: NavigationBar(
        selectedIndex: _index,
        onDestinationSelected: (i) => setState(() => _index = i),
        destinations: [
          for (var i = 0; i < _labels.length; i++)
            NavigationDestination(icon: Icon(_icons[i]), label: _labels[i]),
        ],
      ),
    );
  }
}


// ========================= lib/firebase_options.dart =========================
// DUMMY – flutterfire configure দিয়ে Replace করো
import 'package:firebase_core/firebase_core.dart' show FirebaseOptions;
import 'package:flutter/foundation.dart' show defaultTargetPlatform, TargetPlatform, kIsWeb;
class DefaultFirebaseOptions {
  static FirebaseOptions get currentPlatform {
    if (kIsWeb) return web;
    switch (defaultTargetPlatform) {
      case TargetPlatform.android: return android;
      case TargetPlatform.iOS: return ios;
      default: return android;
    }
  }
  static const FirebaseOptions web = FirebaseOptions(
    apiKey: 'API_KEY', appId: 'APP_ID', messagingSenderId: 'SENDER', projectId: 'PROJECT_ID',
  );
  static const FirebaseOptions android = FirebaseOptions(
    apiKey: 'API_KEY', appId: 'APP_ID', messagingSenderId: 'SENDER', projectId: 'PROJECT_ID',
  );
  static const FirebaseOptions ios = FirebaseOptions(
    apiKey: 'API_KEY', appId: 'APP_ID', messagingSenderId: 'SENDER', projectId: 'PROJECT_ID', iosBundleId: 'com.example.bdccoin',
  );
}


// ========================= lib/core/ai_updater/auto_update.dart =========================
import 'package:firebase_remote_config/firebase_remote_config.dart';

class AiUpdater {
  Future<void> checkForUpdates() async {
    try {
      final rc = FirebaseRemoteConfig.instance;
      await rc.setConfigSettings(RemoteConfigSettings(fetchTimeout: Duration(seconds: 10), minimumFetchInterval: Duration(minutes: 5)));
      await rc.fetchAndActivate();
      final forceUpdate = rc.getBool('force_update');
      if (forceUpdate) {
        // TODO: Prompt user
      }
    } catch (_) {}
  }
}


// ========================= lib/core/security/verify_nid.dart =========================
import 'package:cloud_firestore/cloud_firestore.dart';

Future<bool> verifyNID(String nid) async {
  final snap = await FirebaseFirestore.instance.collection('users').where('nid', isEqualTo: nid).limit(1).get();
  return snap.docs.isEmpty;
}


// ========================= lib/core/security/reset_pin.dart =========================
import 'package:cloud_firestore/cloud_firestore.dart';

Future<void> resetPinUsingNID(String nid, String newPin) async {
  final user = await FirebaseFirestore.instance.collection('users').where('nid', isEqualTo: nid).limit(1).get();
  if (user.docs.isNotEmpty) {
    await FirebaseFirestore.instance.collection('users').doc(user.docs.first.id).update({'pin': newPin});
  } else {
    throw Exception('User not found');
  }
}


// ========================= lib/core/config/remote_config.dart =========================
import 'package:firebase_remote_config/firebase_remote_config.dart';
class AppRemoteConfig {
  static final AppRemoteConfig _i = AppRemoteConfig._();
  AppRemoteConfig._();
  factory AppRemoteConfig() => _i;
  late FirebaseRemoteConfig _rc;
  Future<void> init() async {
    _rc = FirebaseRemoteConfig.instance;
    await _rc.setConfigSettings(RemoteConfigSettings(fetchTimeout: Duration(seconds: 10), minimumFetchInterval: Duration(minutes: 5)));
    await _rc.fetchAndActivate();
  }
  String get adminTransferSecret => _rc.getString('admin_transfer_secret');
}


// ========================= lib/core/utils/logger.dart =========================
void logd(String msg) => print('[BDCCOIN] $msg');


// ========================= lib/shared/providers.dart =========================
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../features/wallet/application/wallet_controller.dart';
import '../features/wallet/infrastructure/wallet_repository.dart';

final walletRepositoryProvider = Provider((ref) => WalletRepository());
final walletControllerProvider = StateNotifierProvider<WalletController, WalletState>((ref) {
  return WalletController(ref.read(walletRepositoryProvider));
});


// ========================= lib/shared/widgets/app_card.dart =========================
import 'package:flutter/material.dart';

class AppCard extends StatelessWidget {
  const AppCard({super.key, required this.child});
  final Widget child;
  @override
  Widget build(BuildContext context) {
    return Card(child: Padding(padding: EdgeInsets.all(16), child: child));
  }
}


// ========================= lib/features/wallet/application/wallet_controller.dart =========================
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../infrastructure/wallet_repository.dart';

class WalletState {
  final double balance;
  final bool loading;
  final String? error;
  WalletState({required this.balance, this.loading = false, this.error});
  WalletState copyWith({double? balance, bool? loading, String? error}) =>
      WalletState(balance: balance ?? this.balance, loading: loading ?? this.loading, error: error);
}

class WalletController extends StateNotifier<WalletState> {
  final WalletRepository _repo;
  WalletController(this._repo) : super(WalletState(balance: 0)) { _init(); }
  Future<void> _init() async {
    state = state.copyWith(loading: true);
    try { state = state.copyWith(balance: await _repo.fetchBalance(), loading: false); }
    catch (e) { state = state.copyWith(error: e.toString(), loading: false); }
  }
  Future<void> send(String to, double amount) async {
    state = state.copyWith(loading: true);
    try { await _repo.send(to: to, amount: amount); state = state.copyWith(balance: await _repo.fetchBalance(), loading: false); }
    catch (e) { state = state.copyWith(error: e.toString(), loading: false); }
  }
}


// ========================= lib/features/wallet/infrastructure/wallet_repository.dart =========================
import 'dart:math';
class WalletRepository {
  double _mockBalance = 10000;
  Future<double> fetchBalance() async { await Future.delayed(Duration(milliseconds: 400)); return _mockBalance; }
  Future<void> send({required String to, required double amount}) async {
    await Future.delayed(Duration(milliseconds: 400));
    if (amount <= 0) throw Exception('Invalid amount');
    if (amount > _mockBalance) throw Exception('Insufficient balance');
    _mockBalance -= amount;
    print('Txn sent to $to: $amount');
  }
}


// ========================= lib/features/wallet/presentation/wallet_screen.dart =========================
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:qr_flutter/qr_flutter.dart';
import '../../../shared/providers.dart';
import '../../../shared/widgets/app_card.dart';
import '../application/wallet_controller.dart';

class WalletScreen extends ConsumerWidget {
  const WalletScreen({super.key});
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final state = ref.watch(walletControllerProvider);
    final controller = ref.read(walletControllerProvider.notifier);
    return ListView(
      padding: EdgeInsets.all(16),
      children: [
        AppCard(
          child: Column(
            crossAxisAlignment: CrossAxisAlignment.start,
            children: [
              Text("BDCCOIN Balance", style: TextStyle(fontSize: 18)),
              SizedBox(height: 8),
              if (state.loading) LinearProgressIndicator(),
              Text("৳ ${state.balance.toStringAsFixed(2)}", style: TextStyle(fontSize: 28, fontWeight: FontWeight.bold)),
              if (state.error != null) Text(state.error!, style: TextStyle(color: Colors.redAccent)),
              SizedBox(height: 16),
              Row(children: [
                ElevatedButton.icon(icon: Icon(Icons.send), label: Text("Send"), onPressed: () => _showSendDialog(context, controller)),
                SizedBox(width: 8),
                OutlinedButton.icon(icon: Icon(Icons.qr_code), label: Text("My QR"), onPressed: () {
                  showDialog(context: context, builder: (_) => AlertDialog(title: Text("Your Wallet QR"), content: QrImageView(data: "wallet_12345", size: 200)));
                }),
              ]),
            ],
          ),
        ),
      ],
    );
  }

  void _showSendDialog(BuildContext context, WalletController controller) {
    final toCtrl = TextEditingController(); final amountCtrl = TextEditingController();
    showDialog(context: context, builder: (_) => AlertDialog(
      title: Text("Send BDCCOIN"),
      content: Column(mainAxisSize: MainAxisSize.min, children: [
        TextField(controller: toCtrl, decoration: InputDecoration(labelText: "To wallet address")),
        TextField(controller: amountCtrl, decoration: InputDecoration(labelText: "Amount"), keyboardType: TextInputType.number),
      ]),
      actions: [
        TextButton(onPressed: () => Navigator.pop(context), child: Text("Cancel")),
        ElevatedButton(onPressed: () async {
          await controller.send(toCtrl.text.trim(), double.tryParse(amountCtrl.text.trim()) ?? 0);
          Navigator.pop(context);
        }, child: Text("Send")),
      ],
    ));
  }
}


// ========================= lib/features/transactions/transaction_screen.dart =========================
import 'package:flutter/material.dart';
import '../../shared/widgets/app_card.dart';

class TransactionScreen extends StatelessWidget {
  const TransactionScreen({super.key});
  @override
  Widget build(BuildContext context) {
    final txs = const [
      {"type": "sent", "amount": 500.0, "to": "xxxxxx", "time": "2025-07-24 10:21"},
      {"type": "recv", "amount": 1000.0, "from": "yyyyyy", "time": "2025-07-24 08:11"},
    ];
    return ListView.builder(
      padding: EdgeInsets.all(16),
      itemCount: txs.length,
      itemBuilder: (_, i) {
        final t = txs[i];
        final isSent = t["type"] == "sent";
        final title = isSent
            ? "৳${t["amount"]} sent to ${t["to"]}"
            : "৳${t["amount"]} received from ${t["from"]}";
        return AppCard(child: ListTile(leading: Icon(isSent ? Icons.north_east : Icons.south_west), title: Text(title), subtitle: Text("${t["time"]}")));
      },
    );
  }
}


// ========================= lib/features/community/community_feed.dart =========================
import 'package:flutter/material.dart';
import '../../shared/widgets/app_card.dart';

class CommunityFeed extends StatelessWidget {
  const CommunityFeed({super.key});
  @override
  Widget build(BuildContext context) {
    final posts = const [
      {"user": "User123", "earned": 450, "likes": 10, "time": "2025-07-24 10:00"},
      {"user": "Anon42", "earned": 1200, "likes": 33, "time": "2025-07-24 09:45"},
    ];
    return ListView(
      padding: EdgeInsets.all(16),
      children: [
        Text("Community Earnings Feed", style: TextStyle(fontSize: 22, fontWeight: FontWeight.bold)),
        SizedBox(height: 12),
        for (final p in posts)
          AppCard(child: ListTile(
            title: Text("${p["user"]} earned ৳${p["earned"]}"),
            subtitle: Text("${p["time"]}"),
            trailing: Row(mainAxisSize: MainAxisSize.min, children: [Icon(Icons.thumb_up_alt_outlined), SizedBox(width: 4), Text("${p["likes"]}")]),
          )),
      ],
    );
  }
}


// ========================= lib/features/charts/tradingview_chart.dart =========================
import 'package:flutter/material.dart';
import 'package:webview_flutter/webview_flutter.dart';

class TradingViewChart extends StatelessWidget {
  const TradingViewChart({super.key});
  @override
  Widget build(BuildContext context) {
    return SafeArea(child: WebView(initialUrl: 'https://www.tradingview.com/chart/?symbol=BDCCOINUSDT', javascriptMode: JavascriptMode.unrestricted));
  }
}


// ========================= lib/features/twitter/twitter_service.dart =========================
class TwitterService {
  Future<void> postFridayAd() async {
    final today = DateTime.now();
    if (today.weekday == DateTime.friday) {
      final content = "🔥 BDCCOIN – Earn, Trade, Secure! Join now!";
      // TODO: Twitter API call
    }
  }
}.
