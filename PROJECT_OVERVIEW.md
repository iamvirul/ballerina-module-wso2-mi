# WSO2 MI Ballerina Module Project Overview

## 1. Introduction

This document provides a comprehensive overview of the `ballerina-module-wso2-mi` project. It is intended for new developers who want to understand the project's architecture, components, and development process.

The project is an SDK that enables developers to use Ballerina for creating transformation logic for WSO2 Micro Integrator (MI). The core mechanism involves a Ballerina annotation, `@mi:Operation`, which developers add to public functions. The project then provides the necessary tooling to validate this Ballerina code and, through a separate process, generate a deployable artifact for WSO2 MI.

## 2. Project Structure

The project is a multi-module Gradle build. The key modules are:

*   `ballerina/`: The Ballerina module (`wso2/mi`) that provides the public `@mi:Operation` annotation.
*   `compiler-plugin/`: A Java-based Ballerina compiler plugin that validates the Ballerina code during compilation.
*   `tests/`: Contains the integration and unit tests for the project.
*   `examples/`: Contains sample projects that demonstrate how to use the module.

## 3. Core Components

### 3.1. Ballerina Module (`wso2/mi`)

The Ballerina module, located in the `ballerina` directory, defines the public API of the project. This is limited to the `public annotation Operation;` definition in `mi_api.bal`. This annotation is the main entry point for developers using this module.

### 3.2. Compiler Plugin

The compiler plugin, located in the `compiler-plugin` directory, is the intellectual core of the project. It contains the logic to validate Ballerina code written for WSO2 MI. It hooks into the Ballerina compilation process and runs several analysis tasks:

*   **`AnnotationAnalysisTask`**: Validates that functions annotated with `@mi:Operation` have parameters and return types that are compatible with WSO2 MI (e.g., `string`, `json`, `xml`).
*   **`ListenerAndServiceDefAnalysisTask`**: Ensures that no `service` or `listener` declarations are present in the Ballerina code, as these are not supported in a WSO2 MI mediation context.
*   **`VariableDeclarationAnalysisTask`**: Prevents the declaration of variables that have a shape similar to a Ballerina `listener` object.
*   **`FunctionAnalysisTask`**: Identifies `public` functions with MI-compatible signatures that are missing the `@mi:Operation` annotation and provides a code action (an IDE quick fix) to add it. This improves the developer experience.

## 4. Build Process and Code Generation

The build process is managed by Gradle. The root `build.gradle` file orchestrates the build of the sub-modules. The `ballerina/build.gradle` file, using the `io.ballerina.plugin`, is responsible for building the Ballerina module.

A key finding of this investigation is that **this project does not handle code generation**. The compiler plugin's sole responsibility is **validation**.

The `README.md` file mentions a `mi-module-gen` command-line tool. This tool is **not** built by this Gradle project. The investigation of the build scripts confirms that there is no logic for building such a tool. The presence of a `handlebars` dependency in the `ballerina/build.gradle` file is a strong indicator that code generation happens via templating, but the tool that uses it is external to this project.

## 5. Known Issues and Improvement Areas

*   **Outdated Documentation**: The main `README.md` is out of sync with the current codebase. It refers to a `mi-module-gen` tool and a `:tool-mi` Gradle project that do not exist. This is a major source of confusion for new developers.
*   **Unclear Code Generation Process**: The single biggest challenge for a new developer is understanding how to generate the final WSO2 MI artifact. Since the tool is not part of this project, the documentation needs to be updated to make this clear.

## 6. Recommendations for New Developers

1.  **Focus on the Compiler Plugin**: The `compiler-plugin` is where the core logic resides. To understand how the validation works, study the analysis tasks in the `compiler-plugin/src/main/java/io/ballerina/stdlib/mi/plugin/` directory.
2.  **Ignore the `mi-module-gen` references in the `README.md`**: As of the current version, this tool is not part of the project. You will need to find and build this tool separately.
3.  **Find the `mi-module-gen` Tool**: Your first task as a new developer should be to locate the source code for the `mi-module-gen` tool. It is likely in another repository within the WSO2 organization. This tool is essential for actually using the output of this project.
4.  **Update the Documentation**: Once you have a clear understanding of the end-to-end process (including the `mi-module-gen` tool), a valuable first contribution would be to update the `README.md` file to reflect the correct process. This would involve removing the incorrect references and adding a clear explanation of the code generation step.
