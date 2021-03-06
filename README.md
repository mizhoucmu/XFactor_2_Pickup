# App Name: Pickup 
![Pickup](https://raw.githubusercontent.com/miworking/XFactor_PickupRecognition/master/app/src/main/res/mipmap-xhdpi/ic_launcher.png)

## What does it do:
Once the app is opened, it will go to the background immediately, so there is no UI for this app.
However it will work in the background as a service, monitoring the user's gesture to decide whether the phone is opened by the owner or not.


## How to kill it:
 Since this app has no UI, and only works as a background service, it can only be killed manually in the settings of the system.
 
 
![Pickup](https://github.com/miworking/XFactor_2_Pickup/blob/master/a.png)
![Kill](https://github.com/miworking/XFactor_2_Pickup/blob/master/b.png)

## How it works
When the screen is turned off, all sensor data will be collected by this service automatically, and these data will be written into a local csv file in external folder like this: /storage/emulated/legacy/Android/data/edu.cmu.ebiz.pickup/files/


Once the screen is turned on, it will read these data from local files, and abstract features from them, and decide whether it is the owner that picked up this phone. 

#### Machine learning algorithm 
Decison is made basing on a random forest model which was trained before, this model file is located in [assets](https://github.com/miworking/XFactor_PickupRecognition/tree/master/app/src/main/assets) folder. And we are using **Random Forest** algorithm prived by [Weka](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&ved=0CB8QFjAAahUKEwjusbKs5bvHAhXKez4KHb80D3M&url=http%3A%2F%2Fwww.cs.waikato.ac.nz%2Fml%2Fweka%2F&ei=VvLXVe6uL8r3-QG_6byYBw&usg=AFQjCNEPpma7O48lI77yyDpwoLXe7vLqHQ&sig2=nmy5t8rpWRtkwt_DOQBD9g&cad=rjt), to use Weka in this app, a [weka.jar](https://github.com/miworking/XFactor_PickupRecognition/blob/master/app/libs/weka.jar) is imported, and this line needs to be added to dependencies in [build.gradle](https://github.com/miworking/XFactor_PickupRecognition/blob/master/app/build.gradle#L27)

`compile files('libs/weka.jar')`

#### Data format and related features
Only ***accelerometer*** and ***magnetic*** sensor data will be used, so there will be two files in this folder: 2015_08_21_23_47_55_acc.csv and 2015_08_21_23_47_55_magetic.csv  ([Why don't use other sensor data?] 
(https://www.dropbox.com/s/bnvwc62nh7kt24q/Pickup.pptx?dl=0) [Page 2])

[104 features](https://www.dropbox.com/s/bnvwc62nh7kt24q/Pickup.pptx?dl=0)will be abstracted
[{Accelerometer, Magnetic} * {X,Y,Z,Magnitude} * {Mean, Std, Min, Max, Percentile25,50,75, FTT[0,1,2]}]


#### Related Classes
All data related classes will be placed in [***pattern***](https://github.com/miworking/XFactor_PickupRecognition/tree/master/app/src/main/java/edu/cmu/ebiz/pickup/pattern) package

- [DoubleFeatureUnit]
(https://github.com/miworking/XFactor_PickupRecognition/blob/master/app/src/main/java/edu/cmu/ebiz/pickup/pattern/DoubleFeatureUnit.java) is the basic class,

- [XYZFeature](https://github.com/miworking/XFactor_PickupRecognition/blob/master/app/src/main/java/edu/cmu/ebiz/pickup/pattern/XYZFeature.java)  has 4 DoubleFeatureUnit members: X,Y,Z,V (Magnitude of X,Y,Z)

- [Feature](https://github.com/miworking/XFactor_PickupRecognition/blob/master/app/src/main/java/edu/cmu/ebiz/pickup/pattern/Feature.java) class contains 2 XYZFeature members: accelerometer and magnetic

- [OnePersonData](https://github.com/miworking/XFactor_PickupRecognition/blob/master/app/src/main/java/edu/cmu/ebiz/pickup/pattern/OnePersonData.java) has a list of Feature as its member, and calculate features from it


#### What's the result?
Once the result is decided from this ***Random Forest*** model, it will be stored in *isOwner* as a boolean variable, and this result will be [sent out as a broadcast to  "org.twinone.locker.pickup.result" Action] (https://github.com/miworking/XFactor_PickupRecognition/blob/master/app/src/main/java/edu/cmu/ebiz/pickup/AEScreenOnOffService.java#L271-L286)

and the XFactor app will register a receiver for this broadcast, and calculate a pickup_score basing on this result.


## Collect data 
This app can also be used to collect data for later modeling.
The only difference is to comment the following [line](https://github.com/miworking/XFactor_2_Pickup/blob/master/app/src/main/java/edu/cmu/ebiz/pickup/AEScreenOnOffService.java#L85
)
so that data will not be cleared everyone when use turns off the screen.
Once data is collected, you can use [TrainModel](https://github.com/miworking/XFactor_3_TrainModel) to preprocess them and generate features in a arff file, so as to be used in Weka. 

After you have collected enough data, you'd better copy this folder to your desktop and rename it so that it wont be erased in the future.
/storage/emulated/legacy/Android/data/edu.cmu.ebiz.pickup/files/ 

Here we [collected data from 6 people](https://www.youtube.com/watch?v=9hVYZ5cTyWo&feature=youtu.be), and put them like [this](https://github.com/miworking/XFactor_3_TrainModel/tree/master/data/pickupData)

you can refer to the README of [TrainModel](https://github.com/miworking/XFactor_3_TrainModel) to go further on how to use these data to traing a model





 
