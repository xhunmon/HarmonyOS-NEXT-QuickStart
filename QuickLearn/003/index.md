# HarmonyOS Learning: Third-Party Open Source Libraries

## Introduction
In HarmonyOS application development, leveraging third-party open source libraries can significantly enhance development efficiency and reduce repetitive work. OpenHarmony Package Manager (OHPM), as the official package management tool, provides developers with abundant third-party library resources covering various functional modules from UI components to business logic processing.

## Official Resources Introduction
The official OHPM repository address is:  This platform aggregates numerous verified open source libraries, supporting categorized search, version management, and one-click integration. It serves as the core channel for HarmonyOS developers to obtain third-party components.

## Detailed Explanation
### Core Functions of OHPM
OHPM is a vital component of the OpenHarmony ecosystem, with main functions including:
1. **Library Resource Management**: Centralized hosting of HarmonyOS-specific open source libraries with standardized version control and dependency resolution.
2. **Project Integration**: Supports quick installation, updates, and removal of libraries via command line or DevEco Studio plugins.
3. **Compatibility Verification**: Ensures library compatibility with specific HarmonyOS versions, reducing integration risks.
4. **Developer Ecosystem**: Encourages community contributions to open source libraries, promoting knowledge sharing and technical iteration.

### Common Types of Open Source Libraries
1. **UI Component Libraries**: e.g., ArkUI UI component library, offering rich pre-built interface elements (tables, calendars, charts, etc.) with multi-device adaptation.
2. **Network Request Libraries**: e.g., ohos-axios, encapsulating HTTP request logic with support for interceptors, timeout control, and automatic JSON parsing.
3. **State Management Libraries**: e.g., ohos-mobx, enabling state sharing between components and simplifying data flow in complex applications.
4. **Utility Libraries**: e.g., ohos-utils, providing common utility functions for date processing, encryption/decryption, and data validation.
5. **Multimedia Libraries**: e.g., ohos-image-loader, supporting image lazy loading, cache management, and format conversion.

### Library Integration and Usage Process
1. **Environment Configuration**: Enable the OHPM plugin in DevEco Studio and configure the repository address (default points to the official repository).
2. **Library Search**: Find target libraries via the OHPM website or IDE search functionality, reviewing documentation and version information.
3. **Installation Command**: Execute `ohpm install library-name@version` to install specific libraries, automatically updating project dependency configurations.

```bash
# Install latest UI component library
ohpm install @ohos/arkui-ui

# Install specific network request library version
ohpm install ohos-axios@1.2.0

# Install development dependencies (e.g., testing tools)
ohpm install --save-dev @ohos/jest
```
4. **Code Reference**: Import library modules in ETS files via `import` statements and call APIs as per documentation.
5. **Version Management**: Use `ohpm update` to upgrade library versions and `ohpm uninstall` to remove unnecessary libraries.

## Practical Advice
1. **Prioritize Officially Recommended Libraries**: "Featured Libraries" on the OHPM homepage undergo rigorous testing for better compatibility and stability.
2. **Monitor Library Activity**: Choose libraries with recent updates and prompt issue responses; avoid long-unmaintained projects.
3. **Control Dependency Quantity**: Excessive dependencies increase package size and compilation time—retain only essential libraries.
4. **Local Cache Management**: Regularly clean redundant files in the `oh_modules` directory using `ohpm cache clean` to free storage space.
5. **Security Audits**: For libraries handling user data, inspect source code for security risks to prevent malicious code introduction.

## Conclusion and Outlook
Third-party open source libraries are crucial for the prosperity of the HarmonyOS ecosystem, and OHPM greatly simplifies library management. As the ecosystem matures, the quantity and quality of open source libraries will continue to improve. Developers should utilize these resources effectively while actively contributing to the community to collectively advance HarmonyOS development efficiency.

## Example: astyle

### 1. Installation and Import
astyle is a HarmonyOS styling solution library that can be quickly integrated via OHPM:
```bash
# Install latest version
ohpm install @qincji/astyle

# Install specific version
ohpm install @qincji/astyle@1.0.0
```

Import in ArkTS files:
```typescript
import { SAdapter, xxx } from '@qincji/astyle';
```

### 2. Basic Usage Example
First, define a custom style by implementing the `SStyle` interface. Only implement methods needed for your scenario:
```typescript
export class xxxStyle implements SStyle<xxxAttribute> {
    initializeModifier(instance?: xxxAttribute): void {
        instance?.backgroundColor('#FFFFFF').setOtherProperties();
    }
    // For other state methods, see the explanation of SState and (SStyle + SIState) correspondence below
}

private style1 = new SAdapter<xxxAttribute>()
    .addStyle(new xxxStyle())
    .addStyle(new xxxStyle())
    .buildAttribute();
    
// CustomTextAdapter is a subclass of SAdapter handling all state styles
private style2 = new CustomTextAdapter().buildAttribute();

// Add styles individually, corresponding to all events of SStyle and parent class
private style7 = new SAdapter<RowAttribute>()
    .addStyle(new ContainStyle())
    .addStyle({
      normalStyle(instance?: RowAttribute | undefined) {
        instance?.backgroundColor('#ff646262');
      }
    })
    .buildAttribute();

// For more usage, see the demo source code   

// Usage: Each object must correspond one-to-one! (Global styles in one line, supporting all system components)
Text('style1 normal & pressed & hover').attributeModifier(this.style1);

Text('style2 CustomTextAdapter').attributeModifier(this.style2);

Row() { }.attributeModifier(this.style7);
```
