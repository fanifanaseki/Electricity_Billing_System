bdccoin_app/
│
├── pubspec.yaml
│
└── lib/
    ├── main.dart
    ├── app.dart
    ├── routes.dart
    ├── firebase_options.dart
    │
    ├── core/
    │   ├── ai_updater/
    │   │   └── auto_update.dart
    │   ├── security/
    │   │   ├── verify_nid.dart
    │   │   └── reset_pin.dart
    │   ├── config/
    │   │   └── remote_config.dart
    │   └── utils/
    │       └── logger.dart
    │
    ├── shared/
    │   ├── providers.dart
    │   └── widgets/
    │       └── app_card.dart
    │
    └── features/
        ├── wallet/
        │   ├── application/
        │   │   └── wallet_controller.dart
        │   ├── infrastructure/
        │   │   └── wallet_repository.dart
        │   └── presentation/
        │       └── wallet_screen.dart
        │
        ├── transactions/
        │   └── transaction_screen.dart
        │
        ├── community/
        │   └── community_feed.dart
        │
        ├── charts/
        │   └── tradingview_chart.dart
        │
        └── twitter/
            └── twitter_service.dart
