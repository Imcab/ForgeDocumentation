# Equals -Forge

## Introduction

The `equals` package in Forge provides robust utilities to help you determine if a robot or field object is "in range" or "at space" in both 2D and 3D environments, or comparing two values, objects. It includes helpful spatial classes and domain checking utilities that are super useful for autonomous logic, alignment, and precision checking.

## Conditional

`Conditional` provides a set of flexible methods for choosing between values based on boolean conditions or predicates. This utility is helpful when making clean, concise decisions without repetitive `if`-`else` blocks.

It supports choosing:

- Between two objects

- Between two primitives

- Among multiple values in a list

### Basic Usage

You can choose between two values using a condition:

``` java

    //Examples
    String winner = Conditional.chooseBetween("Blue", "Red", isBlueAlliance);

    double winnerDouble = Conditional.chooseBetween(1.0, 2.0, isBlueAlliance);

    Pose2d winnerPose = Conditional.chooseBetween(myPose1, myPose2, isBlueAlliance);


```

Or choose based on a `Predicate<T>`:

``` java

    String result = Conditional.chooseBetween("Alpha", "Beta", s -> s.startsWith("A"));

```

### Choosing Among a List

Get an item by index with fallback to the last element:

``` java

    List<String> strategies = List.of("Offense", "Defense", "Support");
    String selected = Conditional.chooseAmong(1, strategies); //"Defense"


```

Find the first element that matches a predicate:

``` java

    String startsWithS = Conditional.chooseAmong(strategies, s -> s.startsWith("S"), "Default"); //"Support"

```

Use a boolean array to choose the first matching condition:

``` java

    boolean[] conditions = new boolean[]{false, true, false};
    String pick = Conditional.chooseAmong(conditions, strategies, "Fallback"); //"Defense"


```

## Epsilon

`Epsilon` helps you compare floating-point numbers (like `double`) safely, using a small tolerance value (aka epsilon). This avoids the usual precision issues you get when comparing doubles directly.

You can use it to:

- Compare two doubles with a custom tolerance

- Use a default precision `(1e-9)` for quick checks

### Usage (with custom epsilon)

Compares two double values using a custom epsilon:

``` java

    boolean areEqual = Epsilon.equals(3.14159, 3.14158, 1e-4); //true

```

This method returns `true` if the absolute difference between the values is less than or equal to the epsilon.

### Usage (default epsilon)

Uses the default kEpsilon value:

``` java

    boolean isSame = Epsilon.equals(5.000000001, 5.000000002); //true

```

Great for when you just want to check if two numbers are “close enough” without thinking too hard.

## Domain

`Domain` is a class that represents a numerical range. It's useful for checking if values are within certain bounds, how close they are to the edges, and enforcing inclusion or exclusion on range limits.

It comes with:

- Enum flags for inclusive (`FULL`) or exclusive (`EMPTY`) edges

- Methods for checking if a value is within, at an edge, or outside the range

- Distance-to-limit calculations

## Creating a Domain

You can create a domain with default full edges:

``` java

    Domain domain = new Domain(0.0, 10.0);

```

Or customize edge inclusivity using `DomainEdge`:

``` java

    Domain domain = new Domain(1.0, Domain.DomainEdge.EMPTY, 5.0, Domain.DomainEdge.FULL);

```

### Checking Range

You can check if a value is inside, outside, or exactly at the half:

``` java

    boolean inside = domain.inRange(5.0); //true
    boolean outside = domain.outOfBounds(11.0); //true
    boolean halfway = domain.atHalf(5.0); //true


```

### Getting Distances

If you want to know how close a value is to either end:

``` java

    double toMin = domain.distanceToMin(3.0); //distance to min
    double toMax = domain.distanceToMax(3.0); //distance to max

```

### Min & Max Value Access

``` java

    double min = domain.minValue();
    double max = domain.maxValue();

```

### Edge-Specific Checks

You can also target edge checks directly:

``` java

    boolean atMin = domain.inMin(0.0);
    boolean atMax = domain.inMax(10.0);

```

### Finding Nearest Edge

``` java

    Domain.DomainLimit nearest = domain.nearest(8.0); //returns MIN or MAX

```

## DomainUtils

`DomainUtils` is a utility class that works alongside `Domain`, providing quick static methods to work with numeric ranges. It helps you determine if a value is within bounds, out of bounds, or how far it is from a specific limit.

### inRange

Check if a value lies between two values (inclusive):

``` java

    boolean inside = DomainUtils.inRange(5.0, 0.0, 10.0); //true

```

### outOfRange

Negated version of inRange:

``` java

    boolean outside = DomainUtils.outOfRange(12.0, 0.0, 10.0); //true


```

### distanceToLimit

Get the absolute distance between a value and a limit:

``` java

    double distance = DomainUtils.distanceToLimit(5.0, 10.0); //5.0

```

Great for determining proximity to either the min or max boundary.

## TwoDimensionalSpace

`TwoDimensionalSpace` defines a 2D region using a center point and a tolerance percentage. You can use it to determine whether a Pose2d lies within this bounded area.

### Constructor

Create a space from coordinates

``` java

    TwoDimensionalSpace space = new TwoDimensionalSpace(2.0, 3.0, 0.1); //10% tolerance

```

Create a space from a Translation2d

``` java

    Translation2d center = new Translation2d(2.0, 3.0);
    TwoDimensionalSpace space = new TwoDimensionalSpace(center, 0.1);

```

### Accessors

Get the center as a Translation2d, Get the tolerance used, Get the coordinate ranges.

``` java

    Translation2d center = space.asTranslation();

    double tol = space.getTolerance();

    Translation2d xRange = space.getXRange(); //xRange.getX() = min, getY() = max
    Translation2d yRange = space.getYRange();

```

### Usage

Check if a `Pose2d` is inside the space

``` java

    boolean inside = space.atSpace(new Pose2d(2.05, 3.05, new Rotation2d()));

```

## ThreeDimensionalSpace

`ThreeDimensionalSpace` defines a bounded 3D region using a center point and a tolerance percentage. It lets you verify whether a `Pose3d` lies within this 3D boundary. This class works exactly the **same** as `TwoDimensionalSpace` but supporting 3d poses.
