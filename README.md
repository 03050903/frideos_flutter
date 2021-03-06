# Frideos-flutter

A collection of helpers, made with the intent to simplify the using of streams and the BLoC pattern, and various widgets for Flutter.

## Helpers for streams and BLoC pattern:

- StreamedValue
- StreamedTransformed
- StreamedList
- StreamedMap
- MemoryValue
- HistoryObject
- TimerObject
- AnimatedObject
- StagedObject
- StreamedSender
- ListSender
- MapSender

## Widgets

- StreamedWidget
- ReceiverWidget
- FuturedWidget
- LinearTransition
- CurvedTransition
- ScrollingText
- Sliders

## [Examples](https://github.com/frideosapps/frideos_flutter/tree/master/examples/frideos_general)

Examples that show how to use this library.

- Streamed objects
- Streamed collections
- TimerObject: a simple stopwatch
- StagedObject
- AnimatedObject
- Multiple selection and tunnel pattern (to share data between two blocs)
- LinearTransition
- CurvedTransition
- Sliders


## Getting started:

Add the library to the dependencies to the pubspec.yaml file:
    
    dependencies:
      ...
      frideos:
        git:
          url: https://github.com/frideosapps/frideos_flutter.git

For the helpers classes import ```frideos_dart.dart```:

```dart
import 'package:frideos/frideos_dart.dart';
```

 ```frideos_flutter.dart``` for the widgets:

```dart
import 'package:frideos/frideos_flutter.dart';
```

```frideos.dart``` if you need both:

```dart
import 'package:frideos/frideos.dart';
```

## Dependencies

- [RxDart](https://pub.dartlang.org/packages/rxdart)


# Helpers

Utility classes to make a little bit easier working with RxDart streams and Flutter.

Since I used the streams from the first time I found them really fun to use so I ended up to use them on every my project. I created a tiny library to make working with streams and Stateless widgets simpler. At first, I wanted something to reduce all the boilerplates to use the streams in the BLoC pattern. The idea was to embed a stream in a class with setters and getters:

```dart
class StreamedValueBase<T> {
  final stream = BehaviorSubject<T>();

  /// timesUpdate shows how many times the got updated
  int timesUpdated = 0;

  /// Sink for the stream
  Function(T) get inStream => stream.sink.add;

  /// Stream getter
  Stream<T> get outStream => stream.stream;

  T get value => stream.value;

  set value(T value) => inStream(value);

  refresh() {
    inStream(value);
  }

  dispose() {
    print('---------- Closing Stream ------ type: $T');
    stream.close();
  }
}
```

Extending this class here is the StreamedValue class:

```dart
class StreamedValue<T> extends StreamedValueBase<T> {
  set value(T value) {
    if (stream.value != value) {
      inStream(value);
      timesUpdated++;
    }
  }
}
```






## StreamedValue

Used in tandem with the StreamedWidget/StreamBuilder, it automatically triggers the refresh of the widget when a new value is set.

This essentially does a simple thing: every time a new value is set, this is compared to the oldest one and if it is different assigned to the variable and sent to stream. Why this? So that when a new value is set, it automatically triggers the StreamerBuilder of the widget and it refreshes without the need to manually add the value to the sink of the stream.

So for example, instead of doing something like this:

```dart
counter += 1;
stream.sink.add(counter);
```

It becomes just:

```dart
counter.value += 1;
```

Then the StreamedValue is used to drive a StreamedWidget/StreamBuilder using the outStream getter.

#### Usage

```dart
// In the BLoC
final counter = StreamedValue<int>();

incrementCounter() {
  counter.value += 2.0;
}


// View
StreamedWidget<int>(
    stream: bloc.count.outStream,
    builder: (BuildContext context,
        AsyncSnapshot<int> snapshot) => Text('Value: ${snapshot.data}',
    noDataChild: Text('NO DATA'),
),
RaisedButton(
    color: buttonColor,
    child: Text('+'),
    onPressed: () {
        bloc.incrementCounter();
        },
),
```

## StreamedTransformed

A special StreamedValue that is used when there is the need to use a StreamTransformer (e.g. validation of input fields).

#### Usage

From the StreamedMap example:

```dart
// In the BLoC class
  final streamedKey = StreamedTransformed<String, int>();



// In the constructor of the BLoC class
  streamedKey.setTransformer(validateKey);



// Validation (e.g. in the BLoC or in a mixin class)
final validateKey =
      StreamTransformer<String, int>.fromHandlers(handleData: (key, sink) {
    var k = int.tryParse(key);
    if (k != null) {
      sink.add(k);
    } else {
      sink.addError('The key must be an integer.');
    }
  });


// In the view:
StreamBuilder(
            stream: bloc.streamedKey.outTransformed,
            builder: (context, AsyncSnapshot<int> snapshot) {
              return Column(
                children: <Widget>[
                  Padding(
                    padding: const EdgeInsets.symmetric(
                      vertical: 12.0,
                      horizontal: 20.0,
                    ),
                    child: TextField(
                      style: TextStyle(
                        fontSize: 18.0,
                        color: Colors.black,
                      ),
                      decoration: InputDecoration(
                        labelText: 'Key:',
                        hintText: 'Insert an integer...',
                        errorText: snapshot.error,
                      ),
                      // To avoid the user could insert text use the TextInputType.number
                      // Here is commented to show the error msg.
                      //keyboardType: TextInputType.number,
                      onChanged: bloc.streamedKey.inStream,
                    ),
                  ),
                ],
              );
            }),
```

## StreamedList

This class has been created to work with lists.

#### Usage

From the StreamedList example:

```dart
  final streamedList = StreamedList<String>();


  // Add to the streamed list the string from the textfield
  addText() {
    streamedList.addElement(streamedText.value);

    // In alternative:
    // streamedList.value.add(streamedText.value);
    // streamedList.refresh(); // To refresh the stream with the new value
  }
```

## StreamedMap

This class has been created to work with maps.

#### Usage

From the streamed map example:

```dart
  final streamedMap = StreamedMap<int, String>();


  // Add to the streamed map the key/value pair put by the user
  addText() {
    var key = int.parse(streamedKey.value);
    var value = streamedText.value;

    streamedMap.value[key] = value;
    streamedMap.refresh();
  }
```

## MemoryValue

The MemoryObject has a property to preserve the previous value. The setter checks for the new value, if it is different from the one already stored, this one is given [oldValue] before storing and streaming the new one.

## HistoryObject

Extends the [MemoryValue] class, adding a [StreamedCollection]. Useful when it is need to store a value in a list.

```dart
final countHistory = HistoryObject<int>();

incrementCounterHistory() {
  countHistory.value++;
}

saveToHistory() {
  countHistory.saveValue();
}
```

## TimerObject

An object that embeds a timer and a stopwatch.

#### Usage

```dart
final timerObject = TimerObject();

startTimer() {
  timerObject.startTimer();
}

stopTimer() {
  timerObject.stopTimer();
}

getLapTime() {
  timerObject.getLapTime();
}

incrementCounter(Timer t) {
  counter.value += 2.0;
}

startPeriodic() {
   var interval = Duration(milliseconds: 1000);
   timerObject.startPeriodic(interval, incrementCounter);
}

```

## AnimatedObject

This class is used to update a value over a period of time. Useful to handle animations using the BLoC pattern.

From the AnimatedObject example:

![AnimatedObject](https://i.imgur.com/jBETLuj.gif)

#### Usage

- #### In the BLoC:

```dart
 // Initial value 0.5, updating interval 20 milliseconds
  final scaleAnimation =
      AnimatedObject<double>(initialValue: 0.5, interval: 20);

  start() {
    scaleAnimation.start(updateScale);
  }

  updateScale(Timer t) {
    scaleAnimation.animation.value += 0.02;

    if (scaleAnimation.animation.value > 5.0) {
      scaleAnimation.reset();
    }
  }

  stop() {
    scaleAnimation.stop();
  }

  reset() {
    scaleAnimation.reset();
  }
```

- #### In the view:

```dart
      Container(
          color: Colors.blueGrey[100],
          child: Column(
            children: <Widget>[
              Container(height: 20.0,),
              StreamedWidget<AnimatedStatus>(
                initialData: AnimatedStatus.stop,
                stream: bloc.scaleAnimation.statusStream,
                builder: (context, AsyncSnapshot<AnimatedStatus> snapshot) {
                  return Row(
                    mainAxisAlignment: MainAxisAlignment.center,
                    children: <Widget>[
                      snapshot.data == AnimatedStatus.active
                          ? RaisedButton(
                              color: Colors.lightBlueAccent,
                              child: Text('Reset'),
                              onPressed: () {
                                bloc.reset();
                              })
                          : Container(),
                      snapshot.data == AnimatedStatus.stop
                          ? RaisedButton(
                              color: Colors.lightBlueAccent,
                              child: Text('Start'),
                              onPressed: () {
                                bloc.start();
                              })
                          : Container(),
                      snapshot.data == AnimatedStatus.active
                          ? RaisedButton(
                              color: Colors.lightBlueAccent,
                              child: Text('Stop'),
                              onPressed: () {
                                bloc.stop();
                              })
                          : Container(),
                    ],
                  );
                },
              ),
              Expanded(
                child: StreamedWidget(
                    stream: bloc.scaleAnimation.animationStream,
                    builder: (context, snapshot) {
                      return Transform.scale(
                          scale: snapshot.data, child: FlutterLogo());
                    }),
              )
            ],
          ),
        ),
```

## StagedObject

A complex class to hadle the rendering of widgets over the time. It takes a collection of "Stages" and triggers the visualization of the widgets at a given time (relative o absolute timing). For example to make a demostration on how to use an application, showing the widgets and pages along with explanations.

![StagedObject](https://i.imgur.com/OaCgxiv.gif)

Every stage is handled by using the Stage class:

```dart
class Stage {
  Widget widget;
  int time; // milliseconds
  Function onShow = () {};
  Stage({this.widget, this.time, this.onShow});
}
```

##### N.B. The onShow callback is used to trigger an action when the stage shows

#### Usage

From the StagedObject example:

1. #### Declare a map <int, Stage>
   Here the map is in the view and is set in the BLoC class by the setStagesMap.

```dart
Map<int, Stage> get widgetsMap => <int, Stage>{
  0: Stage(
      widget: Container(
        width: 200.0,
        height: 200.0,
        color: Colors.indigo[200],
        alignment: Alignment.center,
        key: Key('0'),
        child: ScrollingText(
          text:
            'This stage will last 8 seconds. By the onShow call back it is possibile to assign an action when the widget shows.',
          scrollingDuration: 2000,
          style: TextStyle(
            color: Colors.blue,
            fontSize: 18.0,
            fontWeight: FontWeight.w500)),
        ),
      time: 8000,
      onShow: () {}),
  1: Stage(
      widget: Container(
        width: 200.0,
        height: 200.0,
        color: Colors.indigo[200],
        alignment: Alignment.center,
        key: Key('00'),
        child: ScrollingText(
              text: 'The next widgets will cross      fade.',
              scrollingDuration: 2000,
            ),
          ),
      time: 8000,
      onShow: () {}),

}
```

2. #### In the BLoC

```dart
  final text = StreamedValue<String>();
  final staged = StagedObject();


  // The map can be set through the constructor of the StagedObject
  // or by the setStagesMap method like in this case.
  setMap(Map<int, Stage> widgetsMap) {
    staged.setStagesMap(widgetsMap);
  }


  start() {
    if (staged.getMapLength() > 0) {
      staged.setCallback(sendNextStageText);
      staged.startStages();
    }
  }


  sendNextStageText() {
    var nextStage = staged.getNextStage();
    if (nextStage != null) {
      text.value = "Next stage:";
      widget.value = nextStage.widget;
      stage.value = StageBridge(
          staged.getStageIndex(), staged.getCurrentStage(), nextStage);
    } else {
      text.value = "This is the last stage";
      widget.value = Container();
    }
  }

```

3. #### In the view:

```dart
  // Setting the map in the build method
  StagedObjectBloc bloc = BlocProvider.of(context);
  bloc.setMap(widgetsMap);


  // To show the current widget on the view using the ReceiverWidget.
  // As an alternative it can be used the StreamedWidget/StreamBuilder.
  ReceiverWidget(
    stream: bloc.staged.widgetStream,
  ),
```

# TunnelPattern

Easy pattern to share data between two blocs.

## StreamedSender

Used to make a one-way tunnel beetween two blocs (from blocA to a StremedValue on blocB).

#### Usage

1. #### Define a [StreamedValueBase] derived object in the blocB

```dart
    final receiverStr = StreamedValue<String>();
```

2. #### Define a [StreamedSender] in the blocA

```dart
    final tunnelSenderStr = StreamedSender<String>();
```

3. #### Set the receiver in the sender on the class the holds the instances of the blocs

```dart
    blocA.tunnelSenderStr.setReceiver(blocB.receiverStr);
```

4. #### To send data from blocA to bloc B then:

```dart
    tunnelSenderStr.send("Text from blocA to blocB");
```

## ListSender and MapSender

Like the StreamedSender, but used with collections.

#### Usage

1. #### Define a [StreamedList] or [StreamedMap]object in the blocB

```dart
    final receiverList = StreamedList<int>();
    final receiverMap = StreamedMap<int, String>();
```

2. #### Define a [ListSender]/[MapSender] in the blocA

```dart
    final tunnelList = ListSender<int>();
    final tunnelMap = MapSender<int, String>();
```

3. #### Set the receiver in the sender on the class the holds the instances of the blocs

```dart
    blocA.tunnelList.setReceiver(blocB.receiverList);
    blocA.tunnelMap.setReceiver(blocB.receiverMap);
```

4. #### To send data from blocA to bloc B then:

```dart
    tunnelList.send(list);
    tunnelMap.send(map);
```

# Widgets

## StreamedWidget

#### Usage

```dart
StreamedWidget<String>(stream: stream, builder: (BuildContext context, AsyncSnapshot<String> snasphot)
  => Text(snasphot.data),
  noDataChild: // Widget to show when the stream has no data
  onNoData: () => // or Callback
  errorChild: // Widget to show on error
  onError: (error) => // or Callback
)
```

N.B. The callback is executed only if the respective child is not provided.

## ReceiverWidget

Used with a StreamedValue when the type is a widget to directly stream a widget to the view. Under the hood a StreamedWidget handles the stream and shows the widget.

#### Usage

```dart
ReceiverWidget(stream: streamedValue.outStream),
```

## FuturedWidget

It's a wrapper for the FutureBuilder that gives the possibility to choose directly in the widget the widget to show on waiting the future is resolving or in case of errore or, in alternative, to use the relative callbacks.

#### Usage

```dart
FuturedWidget<String>(future: future, builder: (BuildContext context, AsyncSnapshot<String> snasphot)
  => Text(snasphot.data),
  waitingChild: // Widget to show on waiting
  onWaiting: () => // or Callback
  errorChild: // Widget to show on error
  onError: (error) => // or Callback
)
```

N.B. The callback is executed only if the respective child is not provided.

## LinearTransition

Linear cross fading transition between two widgets, it can be used with the [StagedObject].

![LinearTransition](https://i.imgur.com/mqB3pRA.gif)

#### Usage

```dart
LinearTransition(
  firstWidget: Container(height: 100.0, width: 100.0,
        color: Colors.blue),
  secondWidget: Container(height: 100.0, width: 100.0,
        color: Colors.lime),
  transitionDuration: 4000,
),
```

## CurvedTransition

Cross fading transition between two widgets. This uses the Flutter way to make an animation.

![CurvedTransition](https://i.imgur.com/HMa4mwW.gif)

#### Usage

```dart
CurvedTransition(
  firstWidget: Container(height: 100.0, width: 100.0,
     color: Colors.blue),
  secondWidget: Container(height: 100.0, width: 100.0,
     color: Colors.lime),
  transitionDuration: 4000,
  curve: Curves.bounceInOut,
),
```

## ScrollingText

#### Usage

```dart
ScrollingText(
 text: 'Text scrolling (during 8 seconds).',
 scrollingDuration: 2000, // in milliseconds
 style: TextStyle(color: Colors.blue,
    fontSize: 18.0, fontWeight: FontWeight.w500),
),
```

## Sliders

![Sliders](https://i.imgur.com/H16VE01.gif)

#### Usage

```dart
HorizontalSlider(
  key: _horizontalSliderKey,
  rangeMin: 0.0,
  rangeMax: 3.14,
  //step: 1.0,
  initialValue: bloc.initialAngle,
  backgroundBar: Colors.indigo[50],
  foregroundBar: Colors.indigo[500],
  triangleColor: Colors.red,
  onSliding: (slider) {
    bloc.horizontalSlider(slider);
  },



VerticalSlider(
  key: _verticalSliderKey,
  rangeMin: 0.5,
  rangeMax: 5.5,
  step: 1.0, // Default value 1.0
  initialValue: bloc.initialScale,
  backgroundBar: Colors.indigo[50],
  foregroundBar: Colors.indigo[500],
  triangleColor: Colors.red,
  onSliding: (slider) {
    bloc.verticalSlider(slider);
    },
```
