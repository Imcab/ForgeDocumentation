# Field - forge

## Introduction

Field comes with three classes for you to use:

- `FieldObject` allows you creating (length and width)
- `Pose2DFlipper` for flipping blue alliance poses into red ones based on a `FieldObject`
- `AllianceUtil` for getting the robot's alliance

## FieldObject

For creating a new field object you must pass your length and width in the unit in meters

``` java

    FieldObject field = new FieldObject(Units.inchesToMeters(690.876), Units.inchesToMeters(317));

```

Or you can just get the lastest field for its name:

``` java

    FieldObject field = FieldObject.REEFSCAPE;

```

## Pose2DFlipper

You can use this class to flip your poses into the red alliance, passing it your pose and a `FieldObject` for it to work:

``` java

    Pose2d myPose = new Pose2d(0.6,2.5, new Rotation2d());

    //Flipping a pose based on the lastest field
    Pose2d flippedPose = Pose2DFlipper.flip(myPose, FieldObject.REEFSCAPE);

```

## AllianceUtil

You can get alliance info such as `isBlue`, `isRed`

``` java

    boolean isBlue = AllianceUtil.isBlue();

```
