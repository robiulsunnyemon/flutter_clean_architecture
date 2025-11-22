Flutter এবং Provider ব্যবহার করে Clean Architecture বোঝাটা প্রথমদিকে একটু কঠিন মনে হতে পারে, কিন্তু একবার বুঝে গেলে বড় প্রজেক্ট সামলানো খুব সহজ হয়ে যায়।

আসুন, একটি **ই-কমার্স অ্যাপের (যেমন: দারাজ বা অ্যামাজন)** উদাহরণ দিয়ে, বিশেষ করে **"প্রোডাক্ট লিস্ট দেখানো"**-র মাধ্যমে এই তিনটি লেয়ার খুব সহজভাবে বুঝি।

Clean Architecture-এর মূল মন্ত্র হলো: **Separation of Concerns**। অর্থাৎ, অ্যাপের লজিক, ডেটা এবং ডিজাইন আলাদা আলাদা থাকবে।

নিচে লেয়ারগুলো ভেতর থেকে বাইরের দিকে সাজানো হলো:

-----

### ১. Domain Layer (দ্য বস / নিয়ম-কানুন)

এটা হলো অ্যাপের **ব্রেইন বা মস্তিষ্ক**। এই লেয়ারটি জানে না ডেটা ইন্টারনেট থেকে আসছে নাকি ডাটাবেস থেকে, আবার এটাও জানে না UI দেখতে কেমন হবে। এটি শুধু **"কি করতে হবে"** (Business Logic) সেটা ঠিক করে। এটি সম্পূর্ণ **Pure Dart Code** দিয়ে লেখা হয় (কোনো Flutter প্যাকেজ বা Widget এখানে থাকে না)।

একটি ই-কমার্স অ্যাপে এখানে ৩টি প্রধান জিনিস থাকে:

  * **Entities:** এটি হলো আমাদের মূল অবজেক্ট।
      * *উদাহরণ:* `Product` ক্লাস, যার মধ্যে শুধু `id`, `name`, `price`, `imageUrl` থাকবে। কোনো JSON পার্সিং লজিক এখানে থাকবে না।
  * **Repositories (Interface/Contract):** এটি একটি চুক্তিপত্র। ডোমেইন লেয়ার বলে দেয়, "আমার প্রোডাক্ট ডেটা লাগবে," কিন্তু সে জানে না কোথা থেকে আসবে।
      * *উদাহরণ:* `ProductRepository` নামে একটি abstract class, যেখানে `getProducts()` নামে একটি ফাংশন সিগনেচার থাকবে।
  * **Use Cases:** একেকটি নির্দিষ্ট কাজ।
      * *উদাহরণ:* `GetProductsUseCase`। যখন ইউজার প্রোডাক্ট পেজে যাবে, তখন এই UseCase-টি কল হবে। এর কাজ হলো Repository থেকে ডেটা এনে দেওয়া।

> **সহজ কথায়:** ডোমেইন লেয়ার হলো রেস্টুরেন্টের **ম্যানেজার**, যে শুধু অর্ডার নেয় এবং নিয়ম ঠিক করে, কিন্তু রান্না বা পরিবেশন করে না।

-----

### ২. Data Layer (দ্য ওয়ার্কার / ডেটা সোর্স)

এই লেয়ারের কাজ হলো **"কিভাবে ডেটা আনা হবে"** সেটা ঠিক করা। এটি ডোমেইন লেয়ারের হুকুম তামিল করে।

এখানেও ৩টি প্রধান অংশ থাকে:

  * **Models:** ডোমেইন লেয়ারের `Entity`-র একটি এক্সটেনশন। এখানে JSON থেকে ডেটা কনভার্ট করার লজিক (`fromJson`, `toJson`) থাকে।
      * *উদাহরণ:* `ProductModel` যা `Product` এনটিটিকে extend করে।
  * **Data Sources:** এরা সরাসরি ডেটার সাথে কথা বলে।
      * *Remote Data Source:* API বা Firebase থেকে ডেটা আনে।
      * *Local Data Source:* ফোনের মেমোরি বা SQLite ডাটাবেস থেকে ডেটা আনে।
  * **Repositories (Implementation):** ডোমেইন লেয়ারের সেই `ProductRepository` ইন্টারফেসটি এখানে ইমপ্লিমেন্ট করা হয়। এটি সিদ্ধান্ত নেয় এখন ইন্টারনেট আছে কিনা—থাকলে API থেকে ডেটা আনবে, না থাকলে ক্যাশ (Local DB) থেকে দেখাবে।

> **সহজ কথায়:** ডেটা লেয়ার হলো রেস্টুরেন্টের **কিচেন**, যেখানে আসল রান্নাটা (ডেটা প্রসেসিং) হয়।

-----

### ৩. Presentation Layer (দ্য ফেস / যা ইউজার দেখে)

এটা হলো অ্যাপের **UI বা ডিজাইন**। ইউজার যা দেখে এবং টাচ করে। এখানে আমরা **Provider** ব্যবহার করি স্টেট ম্যানেজ করার জন্য।

এখানে মূলত ২টো জিনিস থাকে:

  * **Provider (State Management):** এটি ডোমেইন লেয়ারের `UseCase` কে কল করে এবং ডেটা পাওয়ার পর UI কে আপডেট করে দেয়।
      * *উদাহরণ:* `ProductProvider`। এটি `isLoading`, `productList`, বা `errorMessage` এর মতো স্টেট ধরে রাখে।
  * **Widgets/Pages:** Flutter এর UI কোড।
      * *উদাহরণ:* `ProductPage` যেখানে `Consumer<ProductProvider>` ব্যবহার করে লোডিং স্পিনার বা প্রোডাক্টের লিস্ট দেখানো হয়।

> **সহজ কথায়:** প্রেজেন্টেশন লেয়ার হলো **ওয়েটার এবং টেবিল**, যেখানে খাবার (ডেটা) সুন্দর করে সাজিয়ে কাস্টমারকে (ইউজার) দেওয়া হয়।

-----

### পুরো ফ্লো (Flow) একনজরে

ধরি, আপনি অ্যাপে ঢুকেছেন এবং প্রোডাক্ট লিস্ট দেখতে চান। তখন পর্দার আড়ালে যা ঘটে:

1.  **Presentation (UI):** `ProductPage` লোড হলো এবং `ProductProvider` কে বলল, "ভাই, প্রোডাক্টগুলো দেখান।"
2.  **Provider:** সে `GetProductsUseCase` কে কল করল।
3.  **Domain (Use Case):** সে `ProductRepository` কে বলল, "আমাকে প্রোডাক্ট লিস্ট দাও।"
4.  **Data (Repository Impl):** সে চেক করল ইন্টারনেট আছে কিনা। তারপর `RemoteDataSource` কে কল করল।
5.  **Data (Source):** API থেকে JSON ডেটা আনল এবং `ProductModel` এ কনভার্ট করে ফেরত পাঠাল।
6.  **Reverse Flow:** ডেটা আবার উল্টো পথে `Data -> Domain -> Provider` হয়ে `UI`-তে চলে আসলো এবং ইউজার প্রোডাক্ট দেখতে পেল।

-----

### ফোল্ডার স্ট্রাকচার কেমন হতে পারে?

Clean Architecture এ আমরা সাধারণত **Feature-based** ফোল্ডার স্ট্রাকচার ব্যবহার করি।

```text
lib/
  └── features/
      └── product/              <-- ফিচার এর নাম
          ├── data/             <-- Data Layer
          │   ├── datasources/  (API calls)
          │   ├── models/       (JSON parsing)
          │   └── repositories/ (Repository Implementation)
          ├── domain/           <-- Domain Layer (Pure Dart)
          │   ├── entities/     (Core objects)
          │   ├── repositories/ (Interfaces)
          │   └── usecases/     (Logic)
          └── presentation/     <-- Presentation Layer
              ├── providers/    (State Management)
              ├── pages/        (Screens)
              └── widgets/      (Small UI components)
```

-----

**সারসংক্ষেপ:**

  * **Domain:** নিয়মকানুন এবং লজিক (কারো উপর নির্ভরশীল না)।
  * **Data:** ডেটা আনা-নেওয়া এবং প্রসেসিং (Domain এর উপর নির্ভরশীল)।
  * **Presentation:** দেখানো এবং ইউজারের সাথে ইন্টারঅ্যাকশন (Domain এর উপর নির্ভরশীল)।

চলুন ক্লিন আর্কিটেকচার ফলো করে একটি ই-কমার্স এপের প্রডাক্ট ফেচ করার বিষয়টি দেখিঃ-
গুলোকে যতটা সম্ভব সহজ রাখব (জটিল Error Handling বা Dartz প্যাকেজ বাদ দিয়ে), যাতে আপনি মূল স্ট্রাকচারটা ধরতে পারেন।

আমাদের লক্ষ্য: **ইন্টারনেট থেকে প্রোডাক্টের লিস্ট ফেচ করে অ্যাপে দেখানো।**

আমরা **Domain → Data → Presentation** এই ক্রমে কোড লিখব। কারণ `Domain` কারোর ওপর নির্ভর করে না, তাই এটি আগে লেখা ভালো।

-----

### ১. Domain Layer (দ্য বস / লজিক)

এখানে কোনো Flutter কোড বা JSON পার্সিং থাকবে না।

**A. Entity (মূল অবজেক্ট):**
প্রথমে ঠিক করি আমাদের প্রোডাক্ট দেখতে কেমন হবে।

```dart
// lib/features/product/domain/entities/product.dart

class Product {
  final int id;
  final String name;
  final double price;

  Product({required this.id, required this.name, required this.price});
}
```

**B. Repository Interface (চুক্তিপত্র):**
আমরা শুধু বলে দেব আমাদের কী দরকার। কিন্তু কোথা থেকে আসবে তা এখানে বলব না।

```dart
// lib/features/product/domain/repositories/product_repository.dart

abstract class ProductRepository {
  Future<List<Product>> getProducts(); // শুধু মেথডের নাম, বডি নেই
}
```

**C. Use Case (নির্দিষ্ট কাজ):**
UI বা Provider সরাসরি রিপোজিটরি কল না করে এই Use Case কে কল করবে।

```dart
// lib/features/product/domain/usecases/get_products_usecase.dart

class GetProductsUseCase {
  final ProductRepository repository;

  GetProductsUseCase(this.repository);

  Future<List<Product>> call() async {
    return await repository.getProducts();
  }
}
```

> **ব্যাখ্যা:** ডোমেইন লেয়ার বলল: "আমার `Product` দরকার, `ProductRepository` এর মাধ্যমে আনব, আর এই কাজটার নাম `GetProductsUseCase`।"

-----

### ২. Data Layer (দ্য ওয়ার্কার / ইমপ্লিমেন্টেশন)

এখানে আমরা ডোমেইন লেয়ারের কথাগুলো বাস্তবে রূপ দেব।

**A. Model (JSON হ্যান্ডলার):**
`Product` এনটিটিকে এক্সটেন্ড করে আমরা JSON পার্সিং যোগ করব।

```dart
// lib/features/product/data/models/product_model.dart

import '../../domain/entities/product.dart';

class ProductModel extends Product {
  ProductModel({required int id, required String name, required double price})
      : super(id: id, name: name, price: price);

  // JSON থেকে ডাটায় রূপান্তর
  factory ProductModel.fromJson(Map<String, dynamic> json) {
    return ProductModel(
      id: json['id'],
      name: json['name'],
      price: (json['price'] as num).toDouble(),
    );
  }
}
```

**B. Data Source (API কল):**
ধরি আমরা সত্যিকারের API কল করছি (এখানে আমি ফেইক ডেটা দিয়ে সিমুলেট করছি)।

```dart
// lib/features/product/data/datasources/product_remote_data_source.dart

import '../models/product_model.dart';

class ProductRemoteDataSource {
  Future<List<ProductModel>> fetchProductsFromApi() async {
    // ধরুন এখানে http.get() কল হচ্ছে...
    await Future.delayed(Duration(seconds: 2)); // ২ সেকেন্ড লোডিং সিমুলেশন

    // API রেসপন্স (JSON)
    final fakeJsonList = [
      {'id': 1, 'name': 'iPhone 14', 'price': 999.0},
      {'id': 2, 'name': 'Samsung S23', 'price': 850.0},
    ];

    return fakeJsonList.map((e) => ProductModel.fromJson(e)).toList();
  }
}
```

**C. Repository Implementation (সেতুবন্ধন):**
ডোমেইন লেয়ারের `ProductRepository` কে এখানে ইমপ্লিমেন্ট করা হবে।

```dart
// lib/features/product/data/repositories/product_repository_impl.dart

import '../../domain/entities/product.dart';
import '../../domain/repositories/product_repository.dart';
import '../datasources/product_remote_data_source.dart';

class ProductRepositoryImpl implements ProductRepository {
  final ProductRemoteDataSource remoteDataSource;

  ProductRepositoryImpl({required this.remoteDataSource});

  @override
  Future<List<Product>> getProducts() async {
    // Data Source থেকে ডেটা এনে ডোমেইন লেয়ারকে দিচ্ছে
    return await remoteDataSource.fetchProductsFromApi();
  }
}
```

> **ব্যাখ্যা:** ডেটা লেয়ার API থেকে JSON এনে সেটাকে `ProductModel` এ কনভার্ট করল এবং ডোমেইন লেয়ারের নিয়ম মেনে সাপ্লাই দিল।

-----

### ৩. Presentation Layer (UI & Provider)

এখন আমরা ডেটা দেখাব।

**A. Provider (State Management):**
এটি UI এবং লজিকের মাঝখানে থাকে।

```dart
// lib/features/product/presentation/providers/product_provider.dart

import 'package:flutter/material.dart';
import '../../domain/entities/product.dart';
import '../../domain/usecases/get_products_usecase.dart';

class ProductProvider extends ChangeNotifier {
  final GetProductsUseCase getProductsUseCase;

  ProductProvider({required this.getProductsUseCase});

  List<Product> _products = [];
  bool _isLoading = false;

  List<Product> get products => _products;
  bool get isLoading => _isLoading;

  Future<void> fetchProducts() async {
    _isLoading = true;
    notifyListeners(); // লোডিং শুরু হলে UI কে জানাও

    try {
      _products = await getProductsUseCase.call(); // UseCase কল হলো
    } catch (e) {
      debugPrint(e.toString());
    }

    _isLoading = false;
    notifyListeners(); // ডেটা লোড শেষ, UI আপডেট করো
  }
}
```

**B. UI (Screen):**
এখানে আমরা `Consumer` ব্যবহার করে ডেটা দেখাব।

```dart
// lib/features/product/presentation/pages/product_page.dart

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import '../providers/product_provider.dart';

class ProductPage extends StatefulWidget {
  @override
  _ProductPageState createState() => _ProductPageState();
}

class _ProductPageState extends State<ProductPage> {
  @override
  void initState() {
    super.initState();
    // পেজ ওপেন হলেই ডেটা ফেচ হবে
    WidgetsBinding.instance.addPostFrameCallback((_) {
      Provider.of<ProductProvider>(context, listen: false).fetchProducts();
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("E-Commerce Products")),
      body: Consumer<ProductProvider>(
        builder: (context, provider, child) {
          if (provider.isLoading) {
            return Center(child: CircularProgressIndicator());
          }

          return ListView.builder(
            itemCount: provider.products.length,
            itemBuilder: (context, index) {
              final product = provider.products[index];
              return ListTile(
                title: Text(product.name),
                subtitle: Text("\$${product.price}"),
                leading: Icon(Icons.shopping_bag),
              );
            },
          );
        },
      ),
    );
  }
}
```

-----

### ৪. সব একসাথে কানেক্ট করা (Dependency Injection)

সব ক্লাস আলাদা আলাদা তৈরি করেছি। এখন `main.dart` এ এদের জোড়া লাগাতে হবে।

```dart
// main.dart

void main() {
  // 1. Data Source তৈরি
  final remoteDataSource = ProductRemoteDataSource();
  
  // 2. Repository তৈরি (Data Source কে ইনজেক্ট করা হলো)
  final repository = ProductRepositoryImpl(remoteDataSource: remoteDataSource);
  
  // 3. Use Case তৈরি (Repository কে ইনজেক্ট করা হলো)
  final useCase = GetProductsUseCase(repository);

  runApp(
    MultiProvider(
      providers: [
        // 4. Provider তৈরি (Use Case কে ইনজেক্ট করা হলো)
        ChangeNotifierProvider(
          create: (_) => ProductProvider(getProductsUseCase: useCase),
        ),
      ],
      child: MaterialApp(
        home: ProductPage(),
      ),
    ),
  );
}
```

-----

### সারসংক্ষেপ: ডেটা আসলে কীভাবে ফ্লো করছে?

1.  **UI:** `ProductPage` লোড হলো -\> `provider.fetchProducts()` কল করল।
2.  **Presentation:** `ProductProvider` লোডিং `true` করে `useCase.call()` ডাকল।
3.  **Domain:** `GetProductsUseCase` বলল `repository.getProducts()`।
4.  **Data:** `ProductRepositoryImpl` বলল `dataSource.fetchApi()`।
5.  **Result:** ডেটা `Data Layer` থেকে `Domain Layer` হয়ে `Provider` এ আসল। `Provider` লোডিং `false` করে UI আপডেট করে দিল।

Clean Architecture এর সুবিধা হলো, কাল যদি আপনি `Provider` বাদ দিয়ে `Bloc` ব্যবহার করতে চান, শুধু **Presentation Layer** পাল্টালেই হবে। Domain বা Data Layer এ হাত দিতে হবে না।

ডিপেনডেন্সি ইনজেকশন (Dependency Injection বা DI) ক্লিন আর্কিটেকচারের প্রাণ। এটা না বুঝলে আগের কোডের `main.dart` অংশটা গোলমেলে মনে হতে পারে।

আসুন খুব সহজ একটি বাস্তব উদাহরণের মাধ্যমে **"Dependency"** এবং **"Injection"** কী তা বুঝি।

-----

### ১. বাস্তব উদাহরণ: রিমোট এবং ব্যাটারি

কল্পনা করুন আপনার একটি টিভির **রিমোট** আছে। রিমোটটি চলার জন্য **ব্যাটারি** প্রয়োজন।

  * এখানে **রিমোট** হলো মেইন ক্লাস।
  * আর **ব্যাটারি** হলো **Dependency** (কারণ ব্যাটারি ছাড়া রিমোট অচল)।

#### ভুল পদ্ধতি (Without Dependency Injection):

ধরুন, কোম্পানি রিমোটটি এমনভাবে বানাল যে **ব্যাটারিটি ভেতরে ঝালাই (Solder) করা**।

  * **সমস্যা:** ব্যাটারি শেষ হলে আপনাকে পুরো রিমোট ফেলে দিতে হবে। আপনি চাইলেও দুরাসেল (Duracell) এর বদলে এনার্জাইজার (Energizer) ব্যাটারি লাগাতে পারবেন না।

#### সঠিক পদ্ধতি (With Dependency Injection):

কোম্পানি রিমোটের পেছনে একটি ফাঁকা স্লট রাখল। আপনি বাজার থেকে আপনার পছন্দমতো ব্যাটারি কিনে এনে স্লটে ভরে দিলেন।

  * **সুবিধা:** আপনি যেকোনো ব্র্যান্ডের ব্যাটারি ব্যবহার করতে পারেন। ব্যাটারি নষ্ট হলে রিমোট বদলানোর দরকার নেই, শুধু ব্যাটারি বদলালেই হবে।

> **Dependency Injection মানে হলো:** রিমোটের ভেতরে ব্যাটারি তৈরি না করে, **বাইরে থেকে ব্যাটারি এনে রিমোটের ভেতর ঢুকিয়ে দেওয়া (Inject করা)।**

-----

### ২. কোডের মাধ্যমে বোঝা

আসুন দেখি কোডে এটা কীভাবে কাজ করে।

#### ক) খারাপ কোড (Hardcoded Dependency)

এখানে `ProductProvider` তার নিজের ভেতরেই `UseCase` তৈরি করে নিচ্ছে।

```dart
class ProductProvider {
  // ভুল: Provider নিজেই UseCase তৈরি করছে।
  // এখন যদি আমি UseCase পরিবর্তন করতে চাই, আমাকে Provider এর কোড ভাঙতে হবে।
  final GetProductsUseCase useCase = GetProductsUseCase(); 

  void loadData() {
    useCase.call();
  }
}
```

এটা হলো সেই **"ঝালাই করা ব্যাটারি"**। এখানে `Provider` এবং `UseCase` একদম শক্তভাবে লেগে আছে (Tight Coupling)।

#### খ) ভালো কোড (Dependency Injection)

এখানে আমরা কনস্ট্রাক্টরের (Constructor) মাধ্যমে বাইরে থেকে `UseCase` দিচ্ছি।

```dart
class ProductProvider {
  final GetProductsUseCase useCase;

  // সঠিক: বাইরে থেকে কেউ একজন UseCase টা পাস করে দেবে (Inject করবে)।
  ProductProvider({required this.useCase}); 

  void loadData() {
    useCase.call();
  }
}
```

এটা হলো **"ব্যাটারি স্লট ওয়ালা রিমোট"**। এখন `main.dart` থেকে আমরা একে নিয়ন্ত্রণ করতে পারি।

-----

### ৩. আমাদের ই-কমার্স প্রজেক্টে এর ব্যবহার

আমাদের প্রজেক্টে একটি চেইন (Chain) বা শিকল ছিল। আসুন দেখি কে কার ওপর নির্ভরশীল (Dependency):

1.  **Provider** কাজ করার জন্য **Use Case**-এর ওপর নির্ভরশীল।
2.  **Use Case** কাজ করার জন্য **Repository**-এর ওপর নির্ভরশীল।
3.  **Repository** কাজ করার জন্য **Data Source (API)**-এর ওপর নির্ভরশীল।

তাই `main.dart` এ আমরা নিচ থেকে ওপরের দিকে কানেকশন দিয়েছিলাম:

```dart
// ১. প্রথমে ব্যাটারি কিনলাম (Data Source)
final remoteDataSource = ProductRemoteDataSource();

// ২. সেই ব্যাটারি দিয়ে রিপোজিটরি চালালাম (Injecting DataSource into Repository)
final repository = ProductRepositoryImpl(remoteDataSource: remoteDataSource);

// ৩. সেই রিপোজিটরি দিয়ে ইউজ-কেস চালালাম (Injecting Repository into UseCase)
final useCase = GetProductsUseCase(repository);

// ৪. সবশেষে সেই ইউজ-কেস প্রোভাইডারে দিলাম (Injecting UseCase into Provider)
final provider = ProductProvider(getProductsUseCase: useCase);
```

### ৪. কেন এই ডিপেনডেন্সি ইনজেকশন এত জরুরি?

এর প্রধান দুটি সুবিধা আছে:

**১. টেস্টিং সহজ হয় (Testing):**
ধরুন আপনি চেক করতে চান `ProductProvider` ঠিকঠাক কাজ করছে কিনা। কিন্তু আপনার ইন্টারনেট নেই বা রিয়েল API কল করতে চান না।
DI ব্যবহার করায়, আপনি সহজেই একটি **নকল (Fake/Mock) UseCase** প্রোভাইডারের মধ্যে ঢুকিয়ে দিয়ে টেস্ট করতে পারবেন।

**২. পরিবর্তন সহজ হয় (Maintainability):**
আজ আপনি `RemoteDataSource` (API) ব্যবহার করছেন। কাল যদি ক্লায়েন্ট বলে, "না, ডেটা এখন ফায়ারবেস (Firebase) থেকে আসবে," তখন আপনাকে পুরো অ্যাপের কোড ঘাঁটতে হবে না।
শুধু নতুন একটা `FirebaseDataSource` বানিয়ে `main.dart` এ ইনজেক্ট করে দিলেই হবে। `Provider` বা `Domain` লেয়ার টেরই পাবে না যে ডেটা সোর্স বদলে গেছে\!

-----

**সারসংক্ষেপ:**
**Dependency** হলো এমন কোনো অবজেক্ট যা ছাড়া আপনার ক্লাস চলতে পারে না। আর **Injection** হলো সেই অবজেক্টটিকে ক্লাসের ভেতরে তৈরি না করে বাইরে থেকে পাস করে দেওয়া।

এখন কি ডিপেনডেন্সি ইনজেকশনের বিষয়টা ক্লিয়ার হয়েছে? চলুন আমি এবার দেখাব  কীভাবে `get_it` প্যাকেজ ব্যবহার করে এই `main.dart` এর কোডগুলো আরও ক্লিন করা যায়।


`get_it` প্যাকেজ ব্যবহার করলে আমাদের `main.dart` ফাইলটি অনেক পরিষ্কার হয়ে যায় এবং ম্যানুয়ালি ডিপেনডেন্সি পাস করার ঝামেলা শেষ হয়ে যায়।

সহজ ভাষায়, `get_it` হলো একটি **স্মার্ট স্টোররুম বা গুদাম (Service Locator)**।
আগে আমরা নিজেরা হাতে করে একটার ভেতর আরেকটা অবজেক্ট ঢোকাতাম। এখন আমরা অ্যাপ চালু হওয়ার শুরুতেই সব অবজেক্ট এই গুদামে সাজিয়ে রাখব। যখন যার যা দরকার হবে, সে শুধু গুদামকে বলবে, "আমাকে `ProductRepository` দাও," আর `get_it` সেটা এনে দেবে।

আসুন ধাপে ধাপে দেখি।

### ধাপ ১: একটি আলাদা ফাইল তৈরি (Injection Container)

আমরা `main.dart` নোংরা করব না। সব ডিপেনডেন্সি সাজানোর জন্য `injection_container.dart` নামে একটা ফাইল বানাব। আমরা সংক্ষেপে `GetIt`-এর ভ্যারিয়েবল নাম দিই `sl` (Service Locator)।

```dart
// lib/injection_container.dart

import 'package:get_it/get_it.dart';
import 'features/product/data/datasources/product_remote_data_source.dart';
import 'features/product/data/repositories/product_repository_impl.dart';
import 'features/product/domain/repositories/product_repository.dart';
import 'features/product/domain/usecases/get_products_usecase.dart';
import 'features/product/presentation/providers/product_provider.dart';

// গ্লোবাল ভ্যারিয়েবল
final sl = GetIt.instance;

Future<void> init() async {
  // --- Features: Product ---

  // 1. Provider (Factory)
  // যখনই কেউ চাইবে, নতুন একটা প্রোভাইডার তৈরি করে দেবে।
  sl.registerFactory(
    () => ProductProvider(getProductsUseCase: sl()), 
    // লক্ষ্য করুন: আমরা ম্যানুয়ালি UseCase বসাচ্ছি না, শুধু sl() কল করছি। 
    // GetIt নিজে বুঝে নেবে এখানে কোন UseCase দরকার।
  );

  // 2. Use Case (Lazy Singleton)
  // অ্যাপে একটাই কপি থাকবে। বারবার নতুন তৈরির দরকার নেই।
  sl.registerLazySingleton(
    () => GetProductsUseCase(sl()), 
    // এখানেও sl() অটোমেটিক Repository খুঁজে এনে দেবে।
  );

  // 3. Repository (Lazy Singleton)
  // ইন্টারফেস এবং ইমপ্লিমেন্টেশন আলাদা হলে এভাবে টাইপ বলে দিতে হয় <Interface>
  sl.registerLazySingleton<ProductRepository>(
    () => ProductRepositoryImpl(remoteDataSource: sl()),
  );

  // 4. Data Source (Lazy Singleton)
  sl.registerLazySingleton<ProductRemoteDataSource>(
    () => ProductRemoteDataSource(),
  );
}
```

### এখানে ২টো গুরুত্বপূর্ণ মেথড ব্যবহার করেছি:

  * **`registerFactory`:** প্রতিবার চাওয়ার সময় **নতুন** অবজেক্ট তৈরি করে। (সাধারণত UI/Provider এর জন্য ব্যবহার হয়, যাতে পেজ রিফ্রেশ হলে নতুন স্টেট পাওয়া যায়)।
  * **`registerLazySingleton`:** অ্যাপ চালু থাকার সময় **একবারই** তৈরি হয় এবং মেমোরিতে থেকে যায়। পরের বার চাইলে সেই পুরনো কপিটাই দেয়। (UseCase, Repository, Data Source এর জন্য ব্যবহার হয় কারণ এগুলো স্টেট ধরে রাখে না, তাই মেমোরি বাঁচানো ভালো)।

-----

### ধাপ ২: ক্লিন `main.dart`

আগে আমাদের `main.dart` এ অনেক লাইন ছিল, এখন দেখুন সেটা কত ছোট হয়ে গেছে\!

```dart
// main.dart

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'injection_container.dart' as di; // di মানে dependency injection
import 'features/product/presentation/pages/product_page.dart';
import 'features/product/presentation/providers/product_provider.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  
  // ১. অ্যাপ চালুর আগে গুদাম (GetIt) সাজিয়ে নিলাম
  await di.init(); 

  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MultiProvider(
      providers: [
        // ২. যাদুটা এখানে! 
        // আমাদের আর লম্বা চেইন বানাতে হচ্ছে না।
        // শুধু di.sl<ProductProvider>() কল করলেই সে তৈরি করা প্রোভাইডার দিয়ে দেবে।
        ChangeNotifierProvider(create: (_) => di.sl<ProductProvider>()),
      ],
      child: MaterialApp(
        title: 'Clean Arch Demo',
        home: ProductPage(),
      ),
    );
  }
}
```

-----

### যাদুটা (Magic) আসলে কোথায় হচ্ছে?

যখন আপনি `di.sl<ProductProvider>()` কল করছেন, তখন `get_it` এর পেছনের লজিকটা অনেকটা এরকম কাজ করে:

1.  `GetIt` তার লিস্ট চেক করে: "আমার কাছে কি `ProductProvider` রেজিস্টার করা আছে?"
2.  হ্যাঁ আছে। সে দেখল এটা বানাতে `GetProductsUseCase` লাগে (`sl()` এর কারণে)।
3.  সে আবার লিস্ট চেক করে: "`GetProductsUseCase` কি আছে?"
4.  হ্যাঁ আছে। সে দেখল এটা বানাতে `ProductRepository` লাগে।
5.  সে আবার চেক করে... এবং `ProductRemoteDataSource` খুঁজে পায়।
6.  সবশেষে নিচ থেকে সব জোড়া লাগিয়ে তৈরি করা `ProductProvider` আপনার হাতে তুলে দেয়।

### সুবিধা কী হলো?

1.  **কোনো চেইন নেই:** `main.dart` ফাইলে আর `datasource -> repo -> usecase` এর লম্বা লাইন লিখতে হলো না।
2.  **অটোমেটিক রেজোলিউশন:** `sl()` অটোমেটিক বুঝে নেয় কার কী দরকার।
3.  **সেন্ট্রাল কন্ট্রোল:** আপনি যদি কখনো `RemoteDataSource` বদলে `LocalDataSource` করতে চান, শুধু `injection_container.dart` ফাইলে এক লাইন চেঞ্জ করলেই পুরো অ্যাপে চেঞ্জ হয়ে যাবে।

আশা করি আপনি  ` get_it` এর ব্যবহার এবং ফ্লো-টা বুজতে পারছেন।
