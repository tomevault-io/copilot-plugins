## dressmeup-ai

> During your interaction with the user, if you find anything reusable in this project (e.g., API endpoints, external services, or data processing methods), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again.

# Instructions

During your interaction with the user, if you find anything reusable in this project (e.g., API endpoints, external services, or data processing methods), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again.

You should also use the `.cursorrules` file as a Scratchpad to organize your thoughts. Especially when you receive a new task, you should first review the content of the Scratchpad, clear old different tasks if necessary, first explain the task, and plan the steps you need to take to complete the task. You can use todo markers to indicate the progress, e.g.
[X] Task 1
[ ] Task 2

Also, update the progress of the task in the Scratchpad when you finish a subtask. Especially when you finish a milestone, it will help improve your depth of task accomplishment to use the Scratchpad to reflect and plan. The goal is to help you maintain a big picture as well as track the progress of the task. Always refer to the Scratchpad when you plan the next step.

# Tools

## Image Processing and API Communication

The app will rely on an external API to process and combine user-uploaded images (e.g., the user's image and an outfit piece like a T-shirt). The following tools are available:

### Image Upload and Processing
- Users upload an image of themselves.
- Users upload an image of an outfit piece (e.g., T-shirt, jacket, dress).
- Both images are sent to an external API for processing.
- The API combines the images and returns the result.
- The processed image is displayed in the app.

### API Request Structure
- The API expects multipart/form-data format with two image inputs.
- The API returns the processed image as a URL or binary data.

### Example API Call (Python)
```python
import requests

url = "https://external-api.com/process"
files = {
    'user_image': open('user_photo.jpg', 'rb'),
    'outfit_image': open('tshirt.png', 'rb')
}
response = requests.post(url, files=files)

if response.status_code == 200:
    result_image_url = response.json().get("processed_image_url")
    print("Processed Image URL:", result_image_url)
else:
    print("Error processing image:", response.text)
```

# Standardized Components

## Image Upload UI
1. Allow users to upload their image.
2. Allow users to upload an outfit image.
3. Display the uploaded images side by side.
4. Provide a "Try On" button to send images to the API.
5. Display the processed image after API response.

## Visual Design
1. Responsive UI layout for both mobile and desktop.
2. Loading indicator while the API processes images.
3. Error handling messages for failed uploads or processing errors.

## Functionality
1. Validate image types (JPEG, PNG) and sizes before upload.
2. Ensure API response is properly handled.
3. Support retry functionality in case of processing failures.

# Lessons

## User-Specified Lessons
- Store user-uploaded images temporarily in memory before sending them to the API.
- Ensure API error handling and proper UI feedback for failed processing requests.
- Use asynchronous requests to prevent UI blocking.
- Validate images before upload to avoid unnecessary API calls.

## Cursor Learned
- Maintain a separate state for original and processed images to prevent accidental overwrites.
- Provide users with an option to re-upload images in case of an incorrect selection.
- Implement proper caching to prevent re-uploading the same images multiple times.

# Scratchpad

## Current Task: Create Outfit Try-On Web App

### Project Setup Steps:
[X] 1. Initialize Next.js project with TypeScript
[X] 2. Set up project structure and dependencies
[X] 3. Create basic UI components
[X] 4. Implement image upload functionality
[X] 5. Integrate with external API
[X] 6. Add error handling and loading states
[X] 7. Implement results page with download/share features
[X] 8. Add responsive styling and UI improvements
[ ] 9. Testing and deployment

### Current Focus:
Setting up the development environment and testing the application.

### Progress:
[X] Project initialization
[X] Dependencies installation
[X] Basic project structure
[X] Component development
[X] API integration
[ ] Testing
[ ] Deployment

### Notes:
- Using Next.js for better performance and SEO
- TypeScript for type safety
- Tailwind CSS for styling
- External API for image processing
- Environment variables for API keys

### Next Steps:
1. Test the application locally
2. Set up proper error handling
3. Add loading states and user feedback
4. Deploy the application

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lolinux2002) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
