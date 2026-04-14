## cursor-working-memory

> Essential guidelines for working with Expo in React Native applications to ensure consistent development practices and optimal performance.

# Expo Development Standards

Essential guidelines for working with Expo in React Native applications to ensure consistent development practices and optimal performance.

<rule>
name: expo_development_standards
description: Standards for Expo development in React Native applications with focus on cross-platform compatibility, performance, and best practices

filters:
  - type: file_extension
    pattern: "\\.tsx?$"
  - type: content
    pattern: "expo|react-native"

actions:
  - type: suggest
    message: |
      ## Expo Best Practices

      ### Platform-Specific Code
      
      1. Use Platform API for conditional code:
         ```typescript
         import { Platform } from 'react-native';
         
         // Preferred approach
         const styles = StyleSheet.create({
           container: {
             paddingTop: Platform.OS === 'ios' ? 20 : 0,
             ...Platform.select({
               ios: { shadowColor: 'black' },
               android: { elevation: 4 },
               default: { /* other platforms */ }
             })
           }
         });
         
         // For larger code blocks
         if (Platform.OS === 'ios') {
           // iOS-specific code
         } else if (Platform.OS === 'android') {
           // Android-specific code
         } else {
           // Web or other platforms
         }
         ```
      
      2. Use platform-specific file extensions:
         - `Component.ios.tsx` - iOS-specific implementation
         - `Component.android.tsx` - Android-specific implementation
         - `Component.web.tsx` - Web-specific implementation
         - `Component.tsx` - Default implementation for all platforms
      
      ### Expo Modules & APIs
      
      1. Import Expo modules correctly:
         ```typescript
         // GOOD: Import from expo-module-name
         import { Camera } from 'expo-camera';
         import * as FileSystem from 'expo-file-system';
         
         // BAD: Import from 'expo'
         import { Camera } from 'expo';
         ```
      
      2. Handle permissions properly:
         ```typescript
         import * as Location from 'expo-location';
         
         const requestLocationPermission = async () => {
           // GOOD: Request permission and handle rejection
           const { status } = await Location.requestForegroundPermissionsAsync();
           
           if (status !== 'granted') {
             // Handle permission denied
             alert('Permission to access location was denied');
             return false;
           }
           
           return true;
         };
         ```
      
      3. Use Expo Constants:
         ```typescript
         import Constants from 'expo-constants';
         
         // Access app config values
         const apiUrl = Constants.expoConfig?.extra?.apiUrl || 'https://default-api.com';
         
         // Get platform information
         const isDevice = Constants.isDevice;
         const deviceName = Constants.deviceName;
         ```
      
      ### Assets Management
      
      1. Use the assets directory structure:
         ```
         assets/
         ├── fonts/          # Custom fonts
         ├── images/         # Images organized by purpose
         │   ├── icons/
         │   ├── backgrounds/
         │   └── logos/
         └── animations/     # Lottie animations
         ```
      
      2. Load assets efficiently:
         ```typescript
         // GOOD: Use require for static assets
         const logoImage = require('../assets/images/logo.png');
         
         // GOOD: Use asset utility for dynamic loading
         import { Asset } from 'expo-asset';
         
         const loadAssets = async () => {
           await Asset.loadAsync([
             require('../assets/images/background.png'),
             require('../assets/images/icon.png')
           ]);
         };
         ```
      
      3. Preload essential assets:
         ```typescript
         import { useFonts } from 'expo-font';
         import * as SplashScreen from 'expo-splash-screen';
         
         // Keep the splash screen visible while assets are loading
         SplashScreen.preventAutoHideAsync();
         
         export default function App() {
           const [fontsLoaded] = useFonts({
             'InterRegular': require('./assets/fonts/Inter-Regular.ttf'),
             'InterBold': require('./assets/fonts/Inter-Bold.ttf'),
           });
           
           const onLayoutRootView = useCallback(async () => {
             if (fontsLoaded) {
               await SplashScreen.hideAsync();
             }
           }, [fontsLoaded]);
           
           if (!fontsLoaded) {
             return null;
           }
           
           return (
             <View onLayout={onLayoutRootView}>
               {/* App content */}
             </View>
           );
         }
         ```
      
      ### Performance Optimization
      
      1. Optimize lists and scrolling:
         ```typescript
         // GOOD: Use optimized list components
         import { FlatList } from 'react-native';
         
         const MyList = ({ data }) => (
           <FlatList
             data={data}
             keyExtractor={(item) => item.id}
             renderItem={({ item }) => <ListItem item={item} />}
             initialNumToRender={10}
             maxToRenderPerBatch={10}
             windowSize={5}
             removeClippedSubviews={true}
           />
         );
         ```
      
      2. Memoize components and callbacks:
         ```typescript
         // GOOD: Memoize complex components
         const MemoizedComponent = React.memo(MyComponent);
         
         // GOOD: Memoize callbacks
         const handlePress = useCallback(() => {
           // Handle press
         }, [dependencies]);
         ```
      
      ### Configuration
      
      1. Use app.json/app.config.js properly:
         ```javascript
         // app.config.js
         export default {
           name: 'MyApp',
           version: '1.0.0',
           // ...other configs
           extra: {
             apiUrl: process.env.API_URL || 'https://api.example.com',
             googleMapsApiKey: process.env.GOOGLE_MAPS_API_KEY
           }
         };
         ```
      
      2. Access configuration correctly:
         ```typescript
         import Constants from 'expo-constants';
         
         const config = {
           apiUrl: Constants.expoConfig?.extra?.apiUrl
         };
         ```
      
      ### Testing
      
      1. Test with Expo's tools:
         ```typescript
         // Use Expo's testing tools
         import { render, fireEvent } from '@testing-library/react-native';
         
         test('button press increments counter', () => {
           const { getByText } = render(<MyComponent />);
           const button = getByText('Increment');
           
           fireEvent.press(button);
           
           expect(getByText('Count: 1')).toBeTruthy();
         });
         ```

examples:
  - input: |
      import React from 'react';
      import { View, Text, StyleSheet, Platform } from 'react-native';
      import Constants from 'expo-constants';
      
      const Header = () => {
        return (
          <View style={styles.container}>
            <Text style={styles.title}>App Name</Text>
          </View>
        );
      };
      
      const styles = StyleSheet.create({
        container: {
          paddingTop: Platform.OS === 'ios' ? 50 : 20,
          backgroundColor: '#fff',
          ...Platform.select({
            ios: {
              shadowColor: '#000',
              shadowOffset: { width: 0, height: 2 },
              shadowOpacity: 0.2,
            },
            android: {
              elevation: 4,
            },
            default: {
              // Other platforms
            }
          })
        },
        title: {
          fontSize: 18,
          fontWeight: 'bold',
        }
      });
      
      export default Header;
    output: "Valid Expo component with proper platform-specific styling"

  - input: |
      import React, { useState } from 'react';
      import { View, Button } from 'react-native';
      import * as ImagePicker from 'expo-image-picker';
      
      const ImagePickerExample = () => {
        const [image, setImage] = useState(null);
        
        const pickImage = async () => {
          // No permission request
          let result = await ImagePicker.launchImageLibraryAsync({
            mediaTypes: ImagePicker.MediaTypeOptions.All,
            quality: 1,
          });
          
          if (!result.cancelled) {
            setImage(result.uri);
          }
        };
        
        return (
          <View>
            <Button title="Pick an image" onPress={pickImage} />
          </View>
        );
      };
      
      export default ImagePickerExample;
    output: "Invalid: Missing permission handling for ImagePicker"

metadata:
  priority: high
  version: 1.0.0
  tags:
    - expo
    - react-native
    - mobile
    - cross-platform
</rule>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/michaelbarone)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/michaelbarone)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
