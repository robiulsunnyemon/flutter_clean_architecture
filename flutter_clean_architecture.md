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
কোডগুলোকে যতটা সম্ভব সহজ রাখব (জটিল Error Handling বা Dartz প্যাকেজ বাদ দিয়ে), যাতে আপনি মূল স্ট্রাকচারটা ধরতে পারেন।

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

