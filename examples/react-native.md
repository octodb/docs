OctoDB on React Native
======================

Please check instructions on [react-native-octodb](https://github.com/octodb/react-native-octodb)


Sample Project
--------------

You can check this working [sample project](http://octodb.io/download/OctoDB-ReactNative-SampleProject.tar.gz)
demonstrating how to use OctoDB on React Native

Download it and extract the files to an empty folder

Then copy and paste these command in the terminal, inside the created folder:

```
yarn  # or npm install
wget http://octodb.io/download/octodb.aar
wget http://octodb.io/download/octodb-free-ios-native-libs.tar.gz
tar zxvf octodb-free-ios-native-libs.tar.gz lib
mv lib node_modules/react-native-octodb/platforms/ios/
mv octodb.aar node_modules/react-native-octodb/platforms/android/
cd ios && pod install && cd ..
```

These will also install the free version of OctoDB (native libraries for Android and iOS)

To run the project:

```
yarn run android
```

Or

```
yarn run ios
```
