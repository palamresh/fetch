# fetch

Main.dart
// ignore_for_file: avoid_print, use_key_in_widget_constructors

import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/material.dart';
import 'package:motivationlapp/splash_service/custom_splash_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();

  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  final Future<FirebaseApp> _initialize = Firebase.initializeApp();

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return FutureBuilder(
        future: _initialize,
        builder: ((context, snapshot) {
          if (snapshot.hasError) {
            print("something is wrong");
          }
          if (snapshot.connectionState == ConnectionState.done) {
            return MaterialApp(
                debugShowCheckedModeBanner: false,
                title: 'MOTIVATION',
                theme: ThemeData(
                  colorScheme: ColorScheme.fromSeed(seedColor: Colors.cyan),
                  useMaterial3: true,
                ),
                // home: SplashScreen(),
                home: const MyCustomSplashScreen());
          }
          return const CircularProgressIndicator();
        }));
  }
}


splash_screen.dart
// ignore_for_file: no_leading_underscores_for_local_identifiers

import 'dart:async';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter/material.dart';
import 'package:motivationlapp/home.dart';

import '../firebase_auth/login_screen.dart';

class MyCustomSplashScreen extends StatefulWidget {
  const MyCustomSplashScreen({super.key});

  @override
  // ignore: library_private_types_in_public_api
  _MyCustomSplashScreenState createState() => _MyCustomSplashScreenState();
}

class _MyCustomSplashScreenState extends State<MyCustomSplashScreen>
    with TickerProviderStateMixin {
  double _fontSize = 2;
  double _containerSize = 1.5;
  double _textOpacity = 0.0;
  double _containerOpacity = 0.0;

  late AnimationController _controller;
  late Animation<double> animation1;
  final auth = FirebaseAuth.instance;

  @override
  void initState() {
    super.initState();

    _controller =
        AnimationController(vsync: this, duration: const Duration(seconds: 3));

    animation1 = Tween<double>(begin: 40, end: 20).animate(CurvedAnimation(
        parent: _controller, curve: Curves.fastLinearToSlowEaseIn))
      ..addListener(() {
        setState(() {
          _textOpacity = 1.0;
        });
      });

    _controller.forward();

    Timer(const Duration(seconds: 2), () {
      setState(() {
        _fontSize = 1.06;
      });
    });

    Timer(const Duration(seconds: 2), () {
      setState(() {
        _containerSize = 2;
        _containerOpacity = 1;
      });
    });

    final user = auth.currentUser;
    if (user != null) {
      Timer(const Duration(seconds: 3), () {
        Navigator.pushReplacement(
            context, MaterialPageRoute(builder: (context) => const HomePage()));
      });
    } else {
      Timer(const Duration(seconds: 3), () {
        Navigator.pushReplacement(context,
            MaterialPageRoute(builder: (context) => const LoginScreen()));
      });
    }
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  // ignore: duplicate_ignore
  Widget build(BuildContext context) {
    double _width = MediaQuery.of(context).size.width;
    double _height = MediaQuery.of(context).size.height;

    return Scaffold(
      backgroundColor: Colors.cyan.shade100,
      body: Stack(
        children: [
          Column(
            children: [
              AnimatedContainer(
                  duration: const Duration(milliseconds: 2000),
                  curve: Curves.fastLinearToSlowEaseIn,
                  height: _height / _fontSize),
              AnimatedOpacity(
                duration: const Duration(milliseconds: 1000),
                opacity: _textOpacity,
                child: Text(
                  'MOTIVATION',
                  style: TextStyle(
                    color: Colors.black,
                    fontWeight: FontWeight.bold,
                    fontSize: animation1.value,
                  ),
                ),
              ),
            ],
          ),
          Center(
            child: AnimatedOpacity(
              duration: const Duration(milliseconds: 2000),
              curve: Curves.fastLinearToSlowEaseIn,
              opacity: _containerOpacity,
              child: AnimatedContainer(
                  duration: const Duration(milliseconds: 2000),
                  curve: Curves.fastLinearToSlowEaseIn,
                  height: _width / _containerSize,
                  width: _width / _containerSize,
                  alignment: Alignment.center,
                  decoration: BoxDecoration(
                    color: Colors.white,
                    borderRadius: BorderRadius.circular(30),
                  ),
                  child: ClipRRect(
                    borderRadius: BorderRadius.circular(30),
                    child: Image.asset(
                      'lib/asset/a2.png',
                      fit: BoxFit.cover,
                    ),
                  )),
            ),
          ),
        ],
      ),
    );
  }
}

HomePage
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';
import 'package:motivationlapp/drawer_page.dart';
import 'package:motivationlapp/english_motivation/en_motivation.dart';

import 'package:motivationlapp/favorite_motivation/favorite_motivation.dart';
import 'package:motivationlapp/goal_motivation/goal_motivation.dart';
import 'package:motivationlapp/hindi_motivation/hindimotivatoinimage.dart';

class HomePage extends StatefulWidget {
  const HomePage({super.key});

  @override
  HomePageState createState() => HomePageState();
}

class HomePageState extends State<HomePage> {
  var currentIndex = 0;
  // ignore: non_constant_identifier_names
  List Page = [
    const HindiMotivation(),
    const GoalMotivation(),
    const EnglishMotivation(),
    const FavoriteMotivation(),
  ];

  // ignore: unused_element
  void _onItemTapped(int index) {
    setState(() {
      currentIndex = index;
    });
  }

  @override
  Widget build(BuildContext context) {
    double displayWidth = MediaQuery.of(context).size.width;
    return WillPopScope(
      onWillPop: () async {
        SystemNavigator.pop();
        return true;
      },
      child: Scaffold(
          // backgroundColor: Colors.cyan.shade100,
          appBar: AppBar(
            toolbarHeight: MediaQuery.of(context).size.height * .08,
            automaticallyImplyLeading: false,
            title: const Text(
              'MOTIVATION ',
              style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold),
            ),
            actions: [
              IconButton(
                  onPressed: () {
                    Navigator.push(
                        context,
                        MaterialPageRoute(
                            builder: (context) => const DrawerPage()));
                  },
                  icon: Icon(
                    Icons.settings,
                    size: displayWidth * .07,
                    color: Colors.black,
                  ))
            ],
            backgroundColor: Theme.of(context).colorScheme.inversePrimary,
            centerTitle: true,
          ),
          bottomNavigationBar: Container(
            margin: EdgeInsets.all(displayWidth * .01),
            height: displayWidth * .155,
            decoration: BoxDecoration(
              color: Colors.white,
              // color: Colors.cyan.shade100,
              boxShadow: [
                BoxShadow(
                  color: Colors.black.withOpacity(.1),
                  blurRadius: 30,
                  offset: const Offset(0, 10),
                ),
              ],
              borderRadius: BorderRadius.circular(50),
            ),
            child: ListView.builder(
              itemCount: 4,
              scrollDirection: Axis.horizontal,
              padding: EdgeInsets.symmetric(horizontal: displayWidth * .02),
              itemBuilder: (context, index) => InkWell(
                onTap: () {
                  setState(() {
                    currentIndex = index;

                    HapticFeedback.lightImpact();
                  });
                },
                splashColor: Colors.transparent,
                highlightColor: Colors.transparent,
                child: Stack(
                  children: [
                    AnimatedContainer(
                      duration: const Duration(seconds: 1),
                      curve: Curves.fastLinearToSlowEaseIn,
                      width: index == currentIndex
                          ? displayWidth * .32
                          : displayWidth * .18,
                      alignment: Alignment.center,
                      child: AnimatedContainer(
                        duration: const Duration(seconds: 1),
                        curve: Curves.fastLinearToSlowEaseIn,
                        height: index == currentIndex ? displayWidth * .12 : 0,
                        width: index == currentIndex ? displayWidth * .32 : 0,
                        decoration: BoxDecoration(
                          color: index == currentIndex
                              ? Colors.cyan.withOpacity(.2)
                              : Colors.transparent,
                          borderRadius: BorderRadius.circular(50),
                        ),
                      ),
                    ),
                    AnimatedContainer(
                      duration: const Duration(seconds: 1),
                      curve: Curves.fastLinearToSlowEaseIn,
                      width: index == currentIndex
                          ? displayWidth * .31
                          : displayWidth * .18,
                      alignment: Alignment.center,
                      child: Stack(
                        children: [
                          Row(
                            children: [
                              AnimatedContainer(
                                duration: const Duration(seconds: 1),
                                curve: Curves.fastLinearToSlowEaseIn,
                                width: index == currentIndex
                                    ? displayWidth * .13
                                    : 0,
                              ),
                              AnimatedOpacity(
                                opacity: index == currentIndex ? 1 : 0,
                                duration: const Duration(seconds: 1),
                                curve: Curves.fastLinearToSlowEaseIn,
                                child: Text(
                                  index == currentIndex
                                      // ignore: unnecessary_string_interpolations
                                      ? '${listOfStrings[index]}'
                                      : '',
                                  style: const TextStyle(
                                    color: Colors.black,
                                    fontWeight: FontWeight.w600,
                                    fontSize: 15,
                                  ),
                                ),
                              ),
                            ],
                          ),
                          Row(
                            children: [
                              AnimatedContainer(
                                duration: const Duration(seconds: 1),
                                curve: Curves.fastLinearToSlowEaseIn,
                                width: index == currentIndex
                                    ? displayWidth * .03
                                    : 20,
                              ),
                              Icon(
                                listOfIcons[index],
                                size: displayWidth * .076,
                                color: index == currentIndex
                                    ? Colors.cyan
                                    : Colors.black26,
                              ),
                            ],
                          ),
                        ],
                      ),
                    ),
                  ],
                ),
              ),
            ),
          ),
          body: Page[currentIndex]),
    );
  }

  List<IconData> listOfIcons = [
    Icons.home_rounded,
    Icons.spa_outlined,
    Icons.format_quote,
    Icons.favorite_rounded,
  ];

  List<String> listOfStrings = [
    'Home',
    'Goals',
    'Quotes',
    'Favorite',
  ];
}


Motivation.dart
// ignore_for_file: sized_box_for_whitespace

import 'package:cached_network_image/cached_network_image.dart';
import 'package:firebase_database/firebase_database.dart';

import 'package:flutter/material.dart';
import 'package:flutter_cache_manager/flutter_cache_manager.dart';
import 'package:motivationlapp/hindi_motivation/h_page.dart';

class HindiMotivation extends StatefulWidget {
  const HindiMotivation({super.key});

  @override
  State<HindiMotivation> createState() => _HindiMotivationState();
}

class _HindiMotivationState extends State<HindiMotivation> {
  final dbref = FirebaseDatabase.instance.ref().child('Motivation');
  final firebaseImageCacheManager = CacheManager(
    Config(
      'firebaseImageCache',
      stalePeriod: const Duration(days: 7), // Cache duration
      maxNrOfCacheObjects: 300, // Maximum number of cached images
    ),
  );
  List list = [];
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Column(
        children: [
          Expanded(
              child: StreamBuilder(
            stream: dbref.child("Motivations").onValue,
            builder: (context, snapshot) {
              if (!snapshot.hasData) {
                return const Center(
                  child: CircularProgressIndicator(),
                );
              } else if (snapshot.connectionState == ConnectionState.none) {
                return const Center(
                  child: Text('Fetch Something'),
                );
              } else if (snapshot.connectionState == ConnectionState.waiting) {
                return const Center(
                  child: CircularProgressIndicator.adaptive(),
                );
              } else if (snapshot.hasError) {
                return const Center(
                  child: Text('Please check your Internet'),
                );
              } else {
                Map<dynamic, dynamic> map =
                    snapshot.data!.snapshot.value as dynamic;

                list.clear();
                list = map.values.toList();
                return Container(
                    margin: const EdgeInsets.all(10.0),
                    child: SingleChildScrollView(
                      physics: const BouncingScrollPhysics(),
                      child: Wrap(spacing: 10.0, runSpacing: 10.0, children: [
                        for (var i = 0;
                            i < snapshot.data!.snapshot.children.length;
                            i++) ...[
                          Container(
                            height: MediaQuery.of(context).size.height * .40,
                            width: MediaQuery.of(context).size.width * .45,
                            child: Stack(
                              fit: StackFit.expand,
                              children: [
                                GestureDetector(
                                  onTap: () {
                                    Navigator.push(
                                        context,
                                        MaterialPageRoute(
                                            builder: (context) =>
                                                HindiPage(user: list[i])));
                                  },
                                  child: ClipRRect(
                                    borderRadius: BorderRadius.circular(20.0),
                                    child: CachedNetworkImage(
                                      imageUrl: list[i]['url'],
                                      fit: BoxFit.cover,
                                      placeholder: (context, url) =>
                                          Image.asset(
                                        'lib/asset/a.jpg',
                                        fit: BoxFit.cover,
                                      ),
                                      errorWidget: (context, url, error) =>
                                          const Center(
                                              child:
                                                  CircularProgressIndicator()),
                                      cacheManager: firebaseImageCacheManager,
                                    ),
                                  ),
                                ),
                                Positioned(
                                    left: 10,
                                    bottom: 3,
                                    child: Text(
                                      list[i]['id'],
                                      style:
                                          const TextStyle(color: Colors.white),
                                    )),
                              ],
                            ),
                          ),
                        ],
                      ]),
                    ));
              }
            },
          )),
        ],
      ),
    );
  }
}

h_post_d.dart
// ignore_for_file: prefer_interpolation_to_compose_strings, no_leading_underscores_for_local_identifiers, avoid_print, sized_box_for_whitespace

import 'dart:io';

import 'package:cached_network_image/cached_network_image.dart';
import 'package:dio/dio.dart';

import 'package:flutter/material.dart';
import 'package:flutter_cache_manager/flutter_cache_manager.dart';

import 'package:http/http.dart' as http;
import 'package:motivationlapp/firebase_service/utils.dart';
import 'package:path_provider/path_provider.dart';
import 'package:permission_handler/permission_handler.dart';

import 'package:share_plus/share_plus.dart';

class HindiPage extends StatefulWidget {
  // ignore: prefer_typing_uninitialized_variables
  final user;

  const HindiPage({super.key, required this.user});

  @override
  State<HindiPage> createState() => _HindiPageState();
}

Color color = Colors.black;
Color color1 = Colors.black;
Color color2 = Colors.black;
bool flag = true;
bool flag1 = true;
bool flag2 = true;

final firebaseImageCacheManager = CacheManager(
  Config(
    'firebaseImageCache',
    stalePeriod: const Duration(days: 7), // Cache duration
    maxNrOfCacheObjects: 100, // Maximum number of cached images
  ),
);

class _HindiPageState extends State<HindiPage> {
  @override
  Widget build(BuildContext context) {
    final Dio dio = Dio();
    Future _requestPermission(Permission permission) async {
      if (await permission.isGranted) {
        return true;
      } else {
        var result = await permission.request();
        if (result == PermissionStatus.granted) {
          return true;
        } else {
          return false;
        }
      }
    }

    Future saveFile(String url, String imagename) async {
      Directory? directory;
      try {
        if (Platform.isAndroid) {
          if (await _requestPermission(Permission.storage)) {
            directory = await getExternalStorageDirectory() as Directory;

            String newpath = "";
            List folders = directory.path.split('/');
            for (int x = 1; x < folders.length; x++) {
              String folder = folders[x];
              if (folder != 'Android') {
                newpath += "/" + folder;
              } else {
                break;
              }
            }
            newpath = newpath + "/Motivation";
            directory = Directory(newpath);
            print(directory.path);
          } else {
            return false;
          }
        } else {
          if (await _requestPermission(Permission.photos)) {
            directory = await getTemporaryDirectory();
          } else {
            return false;
          }
        }
        if (!await directory.exists()) {
          await directory.create(recursive: true);
        }
        if (await directory.exists()) {
          File saveFile = File(directory.path + '/$imagename');
          await dio.download(url, saveFile.path,
              onReceiveProgress: (download, totalsize) {
            setState(() {});
          });
        }
        return true;
      } catch (e) {
        Utils().toastMessage(e.toString());
      }
      return false;
    }

    Future downloadFlle() async {
      setState(() {});
      bool download = await saveFile(widget.user['url'].toString(),
          'hmo${DateTime.now().millisecond}.jpg');
      if (download) {
        print('download file');
        Utils().toastMessage('Image Download Successful');
      } else {
        print("can't download file");
        Utils().toastMessage('Problem  Downloading Image  ');
      }
      setState(() {});
    }

    double _width = MediaQuery.of(context).size.width;
    double _height = MediaQuery.of(context).size.height;

    return Scaffold(
      appBar: AppBar(
        centerTitle: true,
      ),
      body: Container(
        margin: const EdgeInsets.all(10.0),
        height: _height * .80,
        width: _width * .99,
        child: Stack(
          fit: StackFit.expand,
          children: [
            SingleChildScrollView(
              child: Column(
                children: [
                  Container(
                    height: _height * .70,
                    width: _width * .99,
                    child: ClipRRect(
                      borderRadius: BorderRadius.circular(20.0),
                      child: CachedNetworkImage(
                        imageUrl: widget.user['url'],
                        fit: BoxFit.cover,
                        placeholder: (context, url) => Image.asset(
                          'lib/asset/a.jpg',
                          fit: BoxFit.cover,
                        ),
                        errorWidget: (context, url, error) =>
                            const Icon(Icons.error),
                        cacheManager: firebaseImageCacheManager,
                      ),
                    ),
                  ),
                  Card(
                    child: Row(
                      mainAxisAlignment: MainAxisAlignment.spaceBetween,
                      children: [
                        IconButton(
                            onPressed: () {
                              setState(() {
                                if (flag) {
                                  color = Colors.redAccent;
                                  flag = false;
                                } else {
                                  color = Colors.black38;
                                  flag = true;
                                }
                              });
                            },
                            icon: Icon(
                              Icons.favorite,
                              color: color,
                              size: 30,
                            )),
                        IconButton(
                            onPressed: () async {
                              String url = widget.user['url'];
                              final response = await http.get(Uri.parse(url));
                              XFile file = XFile.fromData(response.bodyBytes,
                                  name: 'image.jpg', mimeType: 'image/jpeg');

                              await Share.shareXFiles(
                                [file],
                                text:
                                    "Download Motivational Quotes in Hindi to English Motivational Quotes App \n App for more information ${'https://play.google.com/store/apps/details?id=com.motivationquotesinhinditoenglish.motivationquotes'}",
                              );
                            },
                            icon: const Icon(
                              Icons.share,
                              size: 30,
                            )),
                        IconButton(
                            onPressed: () {
                              downloadFlle();
                            },
                            icon: const Icon(
                              Icons.download,
                              size: 30,
                            )),
                      ],
                    ),
                  )
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}

add_Image.dart
// ignore_for_file: sized_box_for_whitespace, prefer_interpolation_to_compose_strings

import 'dart:io';

import 'package:flutter/foundation.dart';
import 'package:flutter/material.dart';

import 'package:flutter_image_compress/flutter_image_compress.dart';
import 'package:image_picker/image_picker.dart';
import 'package:firebase_storage/firebase_storage.dart' as storage;
import 'package:firebase_database/firebase_database.dart';
import 'package:modal_progress_hud_nsn/modal_progress_hud_nsn.dart';
import 'package:motivationlapp/firebase_service/utils.dart';
import 'package:path_provider/path_provider.dart' as path_provider;

class AddImage extends StatefulWidget {
  const AddImage({super.key});

  @override
  State<AddImage> createState() => _AddImageState();
}

class _AddImageState extends State<AddImage> {
  final mref = FirebaseDatabase.instance.ref().child('Motivation');
  storage.FirebaseStorage store = storage.FirebaseStorage.instance;
  bool showspinner = false;

  File? newImage;

  XFile? image;

  final picker = ImagePicker();

  // method to pick single image while replacing the photo
  Future imagePickerFromGallery() async {
    image = (await picker.pickImage(source: ImageSource.gallery))!;

    final bytes = await image!.readAsBytes();
    final kb = bytes.length / 1024;
    final mb = kb / 1024;

    if (kDebugMode) {
      print('original image size:' + mb.toString());
    }

    final dir = await path_provider.getTemporaryDirectory();
    final targetPath = '${dir.absolute.path}/temp.jpg';

    // converting original image to compress it
    final result = await FlutterImageCompress.compressAndGetFile(
      image!.path,
      targetPath,
      minHeight: 1080, //you can play with this to reduce siz
      minWidth: 1080,
      quality: 70, // keep this high to get the original quality of image
    );
    final data = await result!.readAsBytes();
    final newKb = data.length / 1024;
    final newMb = newKb / 1024;

    if (kDebugMode) {
      print('compress image size:' + newMb.toString());
    }

    newImage = File(result.path);

    setState(() {});
  }

  @override
  Widget build(BuildContext context) {
    return ModalProgressHUD(
      inAsyncCall: showspinner,
      child: Scaffold(
        appBar: AppBar(),
        body: SingleChildScrollView(
          child: Padding(
            padding: const EdgeInsets.all(8.0),
            child: Column(
              children: [
                GestureDetector(
                  onTap: () {
                    //getImage();
                    imagePickerFromGallery();
                  },
                  child: Center(
                    child: Container(
                      height: MediaQuery.of(context).size.height * .6,
                      width: MediaQuery.of(context).size.width * 1,
                      child: image != null
                          ? ClipRRect(
                              borderRadius: BorderRadius.circular(10),
                              child: Image.file(
                                // image!.absolute,
                                // height: 100,
                                // width: 100,
                                fit: BoxFit.cover,
                                File(
                                  newImage!.path,
                                ),
                              ),
                            )
                          : Container(
                              decoration: BoxDecoration(
                                  color: Colors.blue.shade100,
                                  borderRadius: BorderRadius.circular(10)),
                              height: 100,
                              width: 100,
                              child: const Center(
                                child: Icon(
                                  Icons.image,
                                  size: 100,
                                ),
                              ),
                            ),
                    ),
                  ),
                ),
                const SizedBox(
                  height: 20,
                ),
                SizedBox(
                    height: MediaQuery.of(context).size.height * .08,
                    width: MediaQuery.of(context).size.height * .5,
                    child: ElevatedButton(
                        onPressed: () async {
                          setState(() {
                            showspinner = true;
                          });
                          try {
                            int date = DateTime.now().microsecond;
                            storage.Reference ref = storage
                                .FirebaseStorage.instance
                                .ref('/motivation$date');
                            storage.UploadTask uploadTask =
                                ref.putFile(newImage!.absolute);
                            await Future.value(uploadTask);
                            var newUrl = await ref.getDownloadURL();
                            mref
                                .child('Motivations')
                                .child(date.toString())
                                .set({
                              'id': date.toString(),
                              'url': newUrl.toString(),
                            }).then((value) {
                              Utils().toastMessage("Image Upload");
                              setState(() {
                                showspinner = false;
                              });
                              image = null;
                            }).onError((error, stackTrace) {
                              Utils().toastMessage(error.toString());
                              setState(() {
                                showspinner = false;
                              });
                            });
                          } catch (e) {
                            setState(() {
                              showspinner = false;
                            });

                            Utils().toastMessage(e.toString());
                          }
                        },
                        child: const Text('Upload Image')))
              ],
            ),
          ),
        ),
      ),
    );
  }
}

