# BurstPolyExtruder

The original project *by* Nico Reski aims to provide the functionality to create custom meshes (polygons) in Unity based on a collection (array) of vertices directly at runtime. These 2D meshes are created along the x- and z-dimensions in the 3D space. Furthermore, the created custom mesh can be *extruded* (into a 3D prism) along the y-dimension in the 3D space. This project optimizes the performance by replacing Triangle.NET with BurstTriangulator.

#### Background

Some of my research work required me to visualize floods as individual meshes - 2D polygons or 3D prisms - within a VR environment using Unity. Initially, I utilized Unity - PolyExtruder for this purpose. However, performance issues arose due to the use of Triangle.NET for triangulation. To address this, I replaced Triangle.NET with BurstTriangulator, which resulted in significant performance improvements.

## Features

### Triangulation.cs

The `Triangulation.cs` class features a partial implementation of the original [BurstTriangulator](https://github.com/andywiecko/BurstTriangulator) to create render triangles for a custom mesh. The implemented triangulation supports holes in the mesh.

### PolyExtruder.cs

The `PolyExtruder.cs` class is responsible for handling the input data and creating all Unity GameObjects (incl. the actual mesh; 2D polygon / 3D prism) using the features provided through `Triangulation.cs` class in the process. Created 3D prisms (extruded 2D polygons) consist of three GameObjects, namely 1) the bottom mesh (y = 0), 2) the top mesh (y = dynamically assigned; greater than 0), and 3) the surrounding mesh connecting 1 and 2 on their outline accordingly.

Furthermore, the `PolyExtruder.cs` class provides some *quality-of-life* features, such as:

- select whether the mesh should be 2D (polygon) or 3D (extruded, prism)
- calculation of the mesh's (2D polygon's) area
- calculation of the mesh's (2D polygon's) centroid
- set extrusion length ("height")
- select whether or not to visually display the polygon's outline

The main idea is to visualize the 2D input data along the x- and z-dimensions, while the (potential) extrusion is always conducted along the y-dimension. 

### PolyExtruderLight.cs

The `PolyExtruderLight.cs` class is an alternative, lightweight implementation of the original `PolyExtruder.cs` class. Instead of keeping three separate meshes (top, bottom, surround) at runtime, it combines these into one mesh that can be more resource-friendly, particularly when working with many meshes / GameObjects. Please inspect the documentation inside the `PolyExtruderLight.cs` script for additional information and remarks. Please note that the `PolyExtruderLight.cs` class is always setting up the GameObject as an extruded 3D prism.

## Dependencies

**Required:**

- [`Unity.Burst@1.8.18`][burst]
- [`Unity.Collections@2.5.1`][collections]

This project has been built using the following specifications:

* Apple macOS Sequoia 15.3.1
* [Unity](https://unity.com) 6000.0.28f1 Personal (Apple Silicon, LTS).

*Note:* Generally, Unity source code should work also within their Windows counterparts. Please check out the above stated dependencies for troubleshooting.

### Resources

Additional resources used to create this project have been accessed as follows:

* (Original) Triangle library implementation *by* Jonathan Richard Shewchuk ([project web page](http://www.cs.cmu.edu/~quake/triangle.html))
* Triangle.NET *by* Christian Woltering ([CodePlex Archive](https://archive.codeplex.com/?p=triangle), [GitHub snapshot](https://github.com/garykac/triangle.net))
* Using Triangle.NET in Unity ([YouTube video; not available anymore as of 2019-06-04](https://www.youtube.com/watch?v=wByVhzokWPo))
* Determine order (clockwise vs. counter-clockwise) of input vertices ([StackOverflow](https://stackoverflow.com/a/1165943))
* Polygons and meshes *by* Paul Bourke ([project web page](http://paulbourke.net/geometry/polygonmesh/))
* Geo-spatial data about the island of Gotland (Sweden) ([Swedish Statistiska centralbyrån (SCB); accessed 2019-02-06](https://www.scb.se/hitta-statistik/regional-statistik-och-kartor/regionala-indelningar/digitala-granser/))
* Unity - PolyExtruder *by* Nico Reski ([project web page](https://github.com/nicoversity/unity_polyextruder))
* BurstTriangulator *by* Andrzej Więckowski ([project web page](https://github.com/andywiecko/BurstTriangulator))

## How to use

#### Import assets to Unity project

To add the features above to your Unity project, I recommend adding the assets by importing the pre-compiled `BurstPolyExtruder.unitypackage`. Alternatively, the repository directory `src` features a directory titled `Assets/BurstPolyExtruder`, which contains all files that should be in the respective `Assets` directory of an existing Unity project. However, ensure Burst is installed and `unsafe` code is allowed (even when you import the pre-compiled package or add the source code manually). In some rare cases, you should check whether Collections (com.unity.collections) is installed.

#### PolyExtruder.cs class

```cs
// prepare data and options
Vector2[] MyCustomMeshData = new Vector2[]
{
    new Vector2(0.0f, 0.0f),
    new Vector2(10.0f, 0.0f),
    new Vector2(10.0f, 10.0f),
    // ... and more vertices
};
float extrusionHeight = 10.0f;
bool is3D = true;
bool isUsingBottomMeshIn3D = true;
bool isUsingColliders = true;

// create new GameObject (as a child), and further configuration
GameObject polyExtruderGO = new GameObject();
polyExtruderGO.transform.parent = this.transform;
polyExtruderGO.name = "MyCustomMeshName";

// add PolyExtruder script to newly created GameObject,
// keep track of its reference
PolyExtruder polyExtruder = polyExtruderGO.AddComponent<PolyExtruder>();

// configure display of outline (before running the poly extruder)
polyExtruder.isOutlineRendered = true;    // default: false
polyExtruder.outlineWidth = 0.1f;         // default: 0.01f
polyExtruder.outlineColor = Color.blue;   // default: Color.black

// run poly extruder according to input data
polyExtruder.createPrism(polyExtruderGO.name, extrusionHeight, MyCustomMeshData, Color.grey, is3D, isUsingBottomMeshIn3D, isUsingColliders);

// access calculated area and centroid
float area = polyExtruder.polygonArea;
Vector2 centroid = polyExtruder.polygonCentroid;
```

#### Further documentation

All Unity scripts in this project are well documented directly within the source code. Please refer directly to the individual scripts to get a better understanding of the implementation.

### Examples

The imported Unity assets provide three demonstration scenes:

- `Triangulation_Test.unity`, illustrating and testing the implementation of the `Triangulation.cs` class via `TriangulationTest.cs` script.
- `PolyExtruder_Demo.unity`, illustrating the usage of the `PolyExtruder.cs` class via `PolyExtruderDemo.cs` script. The `PolyExtruderDemo.cs` script allows the user to make selections using the Unity Inspector accordingly to a) select an example data set for the custom mesh creation (Triangle, Square, Cross, SCB Kommun RT90 Gotland), b) indicate whether the custom mesh should be created in 2D (polygon) or 3D (prism), c) the length ("height") of the extrusion, and d) whether the extrusion length should be dynamically scaled at runtime (oscillated movement example). 
- `PolyExtruderLight_Demo.unity`, illustrating the usage of the `PolyExtruderLight.cs` class via `PolyExtruderLightDemo.cs` script, following the same examples as featured in the `PolyExtruderDemo.cs` script.

Please refer to these scenes and scripts to learn more about the examples.

### Screenshots - Example Data

Following, some visual impressions of the included example data, visualized using the `Triangulation.cs`, `PolyExtruder.cs`, and `PolyExtruderLight.cs` classes.

#### Triangulation Test: Cross
![Triangulation Test: Cross](docs/test_triangulation_cross.png)

#### PolyExtruder: Cross 3D
![Cross 3D](docs/demo_cross_3D.png)

#### PolyExtruder: Triangle 3D
![Triangle 3D](docs/demo_triangle_3D.png)

#### PolyExtruder: Square 3D
![Square 3D](docs/demo_square_3D.png)

#### PolyExtruder: SCB Kommun RT90 Gotland 3D
![SCB Kommun RT90 Gotland 3D](docs/demo_SCB_Kommun_RT90_Gotland_3D.png)

#### PolyExtruder: SCB Kommun RT90 Gotland 3D with outline
![SCB Kommun RT90 Gotland 3D with outline](docs/demo_SCB_Kommun_RT90_Gotland_3D_outlined.png)

#### PolyExtruder: SCB Kommun RT90 Gotland 3D (movement enabled)
![SCB Kommun RT90 Gotland 3D with movement enabled](docs/demo_SCB_Kommun_RT90_Gotland_3D_movementEnabled.gif)

### Screenshots - SCB Kommun RT90

Following, some visual impressions of the earlier stated use case of visualizing all municipalities in the country of Sweden (see *Background*) using the `PolyExtruder.cs` class. The data has been received from the [Swedish Statistiska centralbyrån (SCB)](https://www.scb.se/hitta-statistik/regional-statistik-och-kartor/regionala-indelningar/digitala-granser/) (accessed: 2019-02-06; **Note:** The data is *not* included as part of this project).

#### PolyExtruder: SCB Kommun RT90 2D
![SCB Kommun RT90 2D](docs/dataNotIncluded_demo_SCB_Kommun_RT90_2D.png)

#### PolyExtruder: SCB Kommun RT90 3D

A random extrusion length ("height") for each municipality has been applied to emphasize the 3D scenario.

![SCB Kommun RT90 3D](docs/dataNotIncluded_demo_SCB_Kommun_RT90_3D.png)

## Known issues

#### Triangulation.cs

1. The function `public static bool triangulate(...)` always returns `true`. In the future, an error list could be implemented to capture errors that occur during the triangulation process.

#### PolyExtruder.cs

1. No holes-support for extrusion (3D prism) is implemented. Although the `Triangulation.cs` script supports holes in the 2D polygon mesh, the support for holes as part of the `PolyExtruder.cs` class *has not* been implemented in this version.

#### PolyExtruderLight.cs

1. No holes-support for extrusion (3D prism) is implemented. Although the `Triangulation.cs` script supports holes in the 2D polygon mesh, the support for holes as part of the `PolyExtruderLight.cs` class *has not* been implemented in this version.
2. Compared to the `PolyExtruder.cs` class, the `Outline Renderer` feature is currently not implemented.
3. Compared to the `PolyExtruder.cs` class, the `MeshCollider` component is currently not implemented.

## Changelog

### 2025-03-15

* First release of BurstPolyExtruder, based on Unity - PolyExtruder (2024-11-25) and BurstTriangulator (v3.6.0).

### 2025-03-31

* Second release of BurstPolyExtruder, based on Unity - PolyExtruder (2024-11-25) and BurstTriangulator (v3.7.0).

### 2025-06-01

* Third release of BurstPolyExtruder, based on Unity - PolyExtruder (2024-11-25) and BurstTriangulator (v3.8.0).

## License
MIT License, see [LICENSE.md](LICENSE.md)

[burst]: https://docs.unity3d.com/Packages/com.unity.burst@1.8/
[collections]: https://docs.unity3d.com/Packages/com.unity.collections@2.5