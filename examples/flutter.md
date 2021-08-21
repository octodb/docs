OctoDB on Flutter
=================

Please check instructions on [octodb_sqflite](https://github.com/octodb/sqflite/tree/master/sqflite)


Sample Project
--------------

You can check this working [sample project](http://octodb.io/download/OctoDB-Flutter-SampleProject.tar.gz)
demonstrating how to use OctoDB on Flutter

Download it and extract the files to an empty folder

Then copy and paste these command in the terminal, inside the created folder:

```
flutter pub get
export FLUTTER_PATH=/path/to/flutter/sdk     # <-- put the path to the Flutter SDK here
echo '----- Android -----'
wget http://octodb.io/download/octodb.aar
mv octodb.aar $FLUTTER_PATH/.pub-cache/hosted/pub.dartlang.org/octodb_sqflite-*/android/
echo '-----   iOS   -----'
mkdir octodb && cd octodb
wget http://octodb.io/download/octodb-free-ios-native-libs.tar.gz
tar zxvf octodb-free-ios-native-libs.tar.gz
cd ..
cd ios && pod install && cd ..
```

These will also install the free version of OctoDB (native libraries for Android and iOS)

To run the project, open the emulator and run:

```
flutter run
```
