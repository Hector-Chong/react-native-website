---
id: pillars-codegen
title: Codegen
---

import NewArchitectureWarning from '../\_markdown-new-architecture-warning.mdx';

<NewArchitectureWarning/>

import Tabs from '@theme/Tabs'; import TabItem from '@theme/TabItem'; import constants from '@site/core/TabsConstants';

The **Codegen** is not a proper pillar, but it is a tool that can be used to avoid writing of a lot of repetitive code. Using the **Codegen** is not mandatory: all the code that is generated by it can also be written manually. However, it generates scaffolding code that could save you a lot of time.

The **Codegen** is invoked automatically by React Native every time an iOS or an Android app is built. Occasionally, you would like to run the scripts that generate the code manually to know which types and files are actually generated: this is a common scenario when developing [**TurboModules**](./pillars-turbomodules) and [**Fabric Components**](./pillars-fabric-components), for example.

This guide teaches how to configure the **Codegen**, how to invoke it manually for each platform, and it describes the generated code.

# Prerequisites

You always need a React Native app to generate the code properly, even when invoking the **Codegen** manually.

The **Codegen** process is tightly coupled with the build of the app and the scripts are located in the `react-native` NPM package.

For the sake of this guide, create a project using the React Native CLI as follows:

```sh
npx react-native init SampleApp --version 0.70.0
```

:::note
This guide assumes that the React Native version in use is 0.70.0 or greater.
Previous versions of React Native uses a version of **Codegen** that requires a slightly different setup.
:::

Then, add the module that requires the **Codegen** as an NPM dependency of the app:

```sh
yarn add <path/to/your/TurboModule_or_FabricComponent>
```

See how to create a [TurboModule](pillars-turbomodules) or a [Fabric Component](pillars-fabric-components) to get more information on how to configure them.

The rest of this guide assumes that you have a `TurboModule` and/or a `Fabric Component` properly set up.

# iOS

## Running the Codegen

The **Codegen** for iOS relies on some Node scripts that are invoked during the build process. The scripts are located in the `MyApp/node_modules/react_native/scripts/` folder.

The script that you have to run is the `generate-artifacts.js` script. This searches among all the dependencies of the app, looking for JS files which respects some specific conventions (look at [TurboModules](pillars-turbomodules) and [Fabric Components](pillars-fabric-components) sections for details) and it generates the required code.

To invoke the script you can run this command from the root folder of your app:

```sh
node node_modules/react_native/scripts/generate-artifacts.js \
    --path SampleApp/ \
    --outputPath <an/output/path> \
```

Given that the app has `TurboModules` and/or `Fabric Components` configured as a dependency, the **Codegen** will look for all of them and it will generate the code in the path you provided.

## The Generated Code

If you run the **Codegen** in your app with an output path of `codegen`, for example, you obtain the following structure:

```title="iOS Codegen output"
codegen
└── build
    └── generated
        └── ios
            ├── MyTurboModuleSpecs
            │   ├── MyTurboModuleSpecs-generated.mm
            │   └── MyTurboModuleSpecs.h
            ├── FBReactNativeSpec
            │   ├── FBReactNativeSpec-generated.mm
            │   └── FBReactNativeSpec.h
            ├── RCTThirdPartyFabricComponentsProvider.h
            ├── RCTThirdPartyFabricComponentsProvider.mm
            └── react
                └── renderer
                    └── components
                        ├── MyFabricComponent
                        │   ├── ComponentDescriptors.h
                        │   ├── EventEmitters.cpp
                        │   ├── EventEmitters.h
                        │   ├── Props.cpp
                        │   ├── Props.h
                        │   ├── RCTComponentViewHelpers.h
                        │   ├── ShadowNodes.cpp
                        │   └── ShadowNodes.h
                        └── rncore
                            ├── ComponentDescriptors.h
                            ├── EventEmitters.cpp
                            ├── EventEmitters.h
                            ├── Props.cpp
                            ├── Props.h
                            ├── RCTComponentViewHelpers.h
                            ├── ShadowNodes.cpp
                            └── ShadowNodes.h
```

The `codegen` folder sits at the root of the hierarchy, as expected. Nested into it there are two more folders: `build/generated`.

Then, there is an `ios` folder which contains:

- A custom folder for each TurboModule.
- The header (`.h`) and implementation (`.mm`) files for the `RCTThirdPartyFabricComponentsProvider`.
- A base `react/renderer/components` folder which contains a custom folder for each `Fabric Component`.

In the example above, there are both a TurboModule and a set of Fabric Components. These are generated by React Native itself: `FBReactNativeSpec` and `rncore`. These modules will always appear even if you don't have any extra TurboModule or Fabric Component: React Native requires them in order to work properly.

### TurboModules

Each TurboModule's folder contains two files: an interface file and an implementation file.

The interface files have the same name of the TurboModule and they contain methods to initialize the JSI interface.

The implementation files, instead, have the `-generated` suffix and they contains the logic to invoke the native methods from JS and viceversa.

### Fabric Components

The content of each Fabric Component folder contains several files. The basic element for a Fabric Componenent is the `ShadowNode`: it represents a node in the React absract tree. The `ShadowNode` represents a React entity, therefore it could need some props, which are defined in the `Props` files and, sometimes, an `EventEmitter`, defined in the corresponding file.

Additionally, the **Codegen** also creates a `ComponentDescriptor.h` and an `RCTComponentViewHelpers.h` files: the first one is used by React Native and Fabric to properly get a reference to the Component, while the latter contains some helper methods and protocols that can be implemented by the Native View to properly respond to JSI invocations.

For further details about how Fabric works, have a look at the [Renderer](/architecture/fabric-renderer) section.

### RCTThirdPartyFabricComponentsProvider

These are an interface and an implementation files for a registry. React Native uses this registry at runtime to retrieve the right class for a required Fabric Component. Once React Native has an handle to that class, it can instantiate it.

# Android

## Running the Codegen

Android `Codegen` relies on a Gradle task to generate the required code. First, you need to configure the Android app to work with the New Architecture, otherwise the Gradle task fails.

1. Open the `MyApp/android/gradle.properties` file.
1. Flip the `newArchEnabled` flag from `false` to `true`.

After that, you can navigate into the `SampleApp/android` folder and run:

```sh
./gradlew generateCodegenArtifactsFromSchema
```

This tasks invokes the `generateCodegenArtifactsFromSchema` on all the the imported projects of the app (the app and all the node modules which are linked to it). It generates the code in the corresponding `node_modules/<dependency>` folder. So, for example, if you have a Fabric Component whose node module is called `my-fabric-component`, the generated code is located in the `SampleApp/node_modules/my-fabric-component/android/build/generated/source/codegen` path.

## The Generated Code

Once the Gradle task completes, you can see different structures for a TurboModule or for a Fabric Component. The following tab shows how they appear:

<Tabs groupId="android-codegen" defaultValue={constants.defaultNewArchFeature} values={constants.newArchFeatures}>
<TabItem value="turbomodules">

```sh
codegen
├── java
│   └── com
│       └── MyTurbomodule
│           └── MyTurbomodule.java
├── jni
│   ├── Android.mk
│   ├── MyTurbomodule-generated.cpp
│   ├── MyTurbomodule.h
│   └── react
│       └── renderer
│           └── components
│               └── MyTurbomodule
│                   ├── ComponentDescriptors.h
│                   ├── EventEmitters.cpp
│                   ├── EventEmitters.h
│                   ├── Props.cpp
│                   ├── Props.h
│                   ├── ShadowNodes.cpp
│                   └── ShadowNodes.h
└── schema.json
```

</TabItem>
<TabItem value="fabric-components">

```sh
codegen
├── java
│   └── com
│       └── facebook
│           └── react
│               └── viewmanagers
│                   ├── MyFabricComponentManagerDelegate.java
│                   └── MyFabricComponentManagerInterface.java
├── jni
│   ├── Android.mk
│   ├── CMakeLists.txt
│   ├── MyFabricComponent-generated.cpp
│   ├── MyFabricComponent.h
│   └── react
│       └── renderer
│           └── components
│               └── MyFabricComponent
│                   ├── ComponentDescriptors.h
│                   ├── EventEmitters.cpp
│                   ├── EventEmitters.h
│                   ├── Props.cpp
│                   ├── Props.h
│                   ├── ShadowNodes.cpp
│                   └── ShadowNodes.h
└── schema.json
```

</TabItem>
</Tabs>

Java can't interoperate seamlessly with C++ as Objective-C++ does. To work properly, the **Codegen** creates some bridging between the Java and the C++ world in the `jni` folder, where the Java Native Interfaces are defined.

Notice that both TurboModules and Fabric Components comes with two build file descriptors: the `Android.mk` and the `CMakeLists.txt`. These are used by the Android app to actually build the external modules.

### TurboModule

The **Codegen** generates a Java abstract class in the `java` package with the same name of the TurboModule. This abstract class has to be implemented by the JNI C++ implementation.

Then, it generates the C++ files in the `jni` folder. They follow the same iOS convention: there is an interface called `MyTurbomodule.h` and an implementation file called `MyTurbomodule-generated.cpp`. The former is an interface that allows React Natvie to initialize the JSI interface for the TurboModule. The latter is the implementation file which contains the logic to invoke the native method from JS and viceversa.

### Fabric Component

The **Codegen** for a Fabric Component contains a `MyFabricComponentManagerInterface.java` and a `MyFabricComponentManagerDelegate.java` in the `java` package. They are implemented and used by the native `MyFabricComponentManager` required to properly load the component at runtime (See the guide on how to create a [Fabric Component](./pillars-fabric-components) for details).

Then, there is a layer of JNI C++ files that are used by Fabric to render the components. The basic element for a Fabric Componenent is the `ShadowNode`: it represents a node in the React absract tree. The `ShadowNode` represents a React entity, therefore it could need some props, which are defined in the `Props` files and, sometimes, an `EventEmitter`, defined in the corresponding file.

The **Codegen** also creates a `ComponentDescriptor.h` which is required to get a proper handle to the Fabric Component.
