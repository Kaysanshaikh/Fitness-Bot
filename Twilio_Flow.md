# 🎨 Twilio Studio Flow Building Guide

Detailed step-by-step instructions for building your WhatsApp fitness bot flows in Twilio Studio.

## 📋 Flow Overview

Your fitness bot will have these main flows:
1. **Welcome Flow** - New user onboarding
2. **Main Menu Flow** - User interaction routing
3. **Workout Flow** - Daily workout delivery
4. **Progress Flow** - Progress tracking and reports
5. **Meeting Flow** - 1-on-1 session scheduling
6. **AI Chat Flow** - Intelligent responses

---

## 🚀 Flow 1: Welcome Flow (New Users)

### **Purpose**: Onboard new users from Google Forms

### **Trigger Configuration**:
- **Trigger Type**: Webhook
- **Event**: onMessageReceived
- **Channel**: WhatsApp

### **Flow Steps**:

#### **Step 1: Check User Status**
```
Widget: Run Function
Function Name: checkUserStatus
Parameters:
  - phoneNumber: {{trigger.message.From}}
  - source: {{trigger.message.Body}}
```

**Function Code**:
```javascript
exports.handler = function(context, event, callback) {
  const phoneNumber = event.phoneNumber;
  
  // Check if user exists in your database
  // For now, we'll use a simple check
  const userExists = false; // Replace with actual database check
  
  callback(null, {
    userExists: userExists,
    phoneNumber: phoneNumber
  });
};
```

#### **Step 2: Split Based on User Status**
```
Widget: Split Based On
Variable: {{widgets.check_user_status.parsed.userExists}}
```

#### **Step 3: Welcome Message (New Users)**
```
Widget: Send Message
To: {{trigger.message.From}}
Body: |
  🏋️‍♀️ Welcome to your AI Fitness Coach!
  
  Hi {{widgets.parse_form_data.parsed.userData.name}}! 👋
  
  I'm excited to be your personal fitness coach! Here's what I can help you with:
  
  💪 Daily workout videos
  📊 Progress tracking
  🥗 Nutrition advice
  📅 1-on-1 coaching sessions
  🎯 Goal setting and motivation
  
  Your first workout will be delivered at {{widgets.parse_form_data.parsed.userData.preferredTime}} today!
  
  Reply with:
  • "Workout" - Get today's workout
  • "Progress" - Check your progress
  • "Help" - See all options
  • "Schedule" - Book a session
```

#### **Step 4: Create User Profile**
```
Widget: Run Function
Function Name: createUserProfile
Parameters:
  - userData: {{widgets.parse_form_data.parsed.userData}}
  - phoneNumber: {{trigger.message.From}}
```

**Function Code**:
```javascript
exports.handler = function(context, event, callback) {
  const userData = JSON.parse(event.userData);
  const phoneNumber = event.phoneNumber;
  
  // Create user profile in your database
  const userProfile = {
    phoneNumber: phoneNumber,
    name: userData.name,
    age: userData.age,
    goals: userData.goals,
    preferredTime: userData.preferredTime,
    experience: userData.experience,
    healthConditions: userData.healthConditions,
    createdAt: new Date().toISOString(),
    currentStreak: 0,
    totalSessions: 0
  };
  
  // Save to database (replace with your database logic)
  console.log('Creating user profile:', userProfile);
  
  callback(null, {
    success: true,
    userProfile: userProfile
  });
};
```

---

## 🎯 Flow 2: Main Menu Flow (Existing Users)

### **Purpose**: Route existing users to appropriate functions

### **Trigger Configuration**:
- **Trigger Type**: Webhook
- **Event**: onMessageReceived
- **Channel**: WhatsApp

### **Flow Steps**:

#### **Step 1: Parse User Intent**
```
Widget: Run Function
Function Name: parseUserIntent
Parameters:
  - message: {{trigger.message.Body}}
  - phoneNumber: {{trigger.message.From}}
```

**Function Code**:
```javascript
exports.handler = function(context, event, callback) {
  const message = event.message.toLowerCase();
  const phoneNumber = event.phoneNumber;
  
  let intent = 'general_question';
  
  if (message.includes('workout') || message.includes('exercise') || message.includes('video')) {
    intent = 'workout_request';
  } else if (message.includes('progress') || message.includes('stats') || message.includes('report')) {
    intent = 'progress_check';
  } else if (message.includes('nutrition') || message.includes('diet') || message.includes('food')) {
    intent = 'nutrition_advice';
  } else if (message.includes('schedule') || message.includes('meeting') || message.includes('book')) {
    intent = 'schedule_meeting';
  } else if (message.includes('help') || message.includes('menu') || message.includes('options')) {
    intent = 'show_menu';
  }
  
  callback(null, {
    intent: intent,
    phoneNumber: phoneNumber,
    originalMessage: event.message
  });
};
```

#### **Step 2: Split Based on Intent**
```
Widget: Split Based On
Variable: {{widgets.parse_intent.parsed.intent}}
```

#### **Step 3: Route to Specific Flows**
```
Widget: Connect to Flow
Flow: Workout Flow (for workout_request)
Flow: Progress Flow (for progress_check)
Flow: Nutrition Flow (for nutrition_advice)
Flow: Meeting Flow (for schedule_meeting)
Flow: AI Chat Flow (for general_question)
```

---

## 💪 Flow 3: Workout Flow

### **Purpose**: Deliver daily workout videos and track completion

### **Trigger**: Connected from Main Menu Flow

### **Flow Steps**:

#### **Step 1: Get Daily Workout**
```
Widget: Run Function
Function Name: getDailyWorkout
Parameters:
  - phoneNumber: {{flow.variables.phoneNumber}}
  - userProfile: {{flow.variables.userProfile}}
```

**Function Code**:
```javascript
exports.handler = function(context, event, callback) {
  const phoneNumber = event.phoneNumber;
  
  // Get user's current day in program
  const currentDay = 1; // Replace with actual database lookup
  
  // Curated workout videos
  const workouts = {
    1: {
      title: "Full Body HIIT Workout",
      videoUrl: "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
      duration: "20 minutes",
      calories: "150-200",
      difficulty: "Beginner",
      equipment: "None"
    },
    2: {
      title: "Strength Training - Upper Body",
      videoUrl: "https://www.youtube.com/watch?v=dQw4w9WgXcQ",
      duration: "25 minutes",
      calories: "180-220",
      difficulty: "Intermediate",
      equipment: "Dumbbells"
    }
    // Add more workouts...
  };
  
  const workout = workouts[currentDay] || workouts[1];
  
  callback(null, {
    workout: workout,
    currentDay: currentDay
  });
};
```

#### **Step 2: Send Workout Message**
```
Widget: Send Message
To: {{flow.variables.phoneNumber}}
Body: |
  🏋️‍♀️ Today's Workout: {{widgets.get_workout.parsed.workout.title}}
  
  ⏱️ Duration: {{widgets.get_workout.parsed.workout.duration}}
  🔥 Calories: {{widgets.get_workout.parsed.workout.calories}}
  📊 Difficulty: {{widgets.get_workout.parsed.workout.difficulty}}
  🏠 Equipment: {{widgets.get_workout.parsed.workout.equipment}}
  
  🎥 Watch here: {{widgets.get_workout.parsed.workout.videoUrl}}
  
  Reply with:
  ✅ "Done" - Mark as completed
  ⏭️ "Skip" - Skip today's workout
  💬 "Feedback" - Share your thoughts
```

#### **Step 3: Wait for Response**
```
Widget: Wait for User Input
Timeout: 7200 (2 hours)
```

#### **Step 4: Handle User Response**
```
Widget: Split Based On
Variable: {{widgets.wait_for_response.parsed.Body}}
```

#### **Step 5: Process Completion**
```
Widget: Run Function (for "Done" response)
Function Name: markWorkoutComplete
Parameters:
  - phoneNumber: {{flow.variables.phoneNumber}}
  - workout: {{widgets.get_workout.parsed.workout}}
```

**Function Code**:
```javascript
exports.handler = function(context, event, callback) {
  const phoneNumber = event.phoneNumber;
  const workout = JSON.parse(event.workout);
  
  // Update user's progress in database
  // Increment streak, total sessions, etc.
  
  callback(null, {
    success: true,
    message: "Great job completing your workout! 💪"
  });
};
```

#### **Step 6: Send Completion Message**
```
Widget: Send Message
To: {{flow.variables.phoneNumber}}
Body: |
  🎉 Amazing work! You've completed today's workout!
  
  ✅ Workout: {{widgets.get_workout.parsed.workout.title}}
  🔥 Calories burned: {{widgets.get_workout.parsed.workout.calories}}
  📈 Your streak: {{widgets.mark_complete.parsed.currentStreak}} days
  
  Keep up the great work! Tomorrow's workout will be even better! 💪
  
  Reply "Progress" to see your stats or "Help" for more options.
```

---

## 📊 Flow 4: Progress Flow

### **Purpose**: Generate and send progress reports

### **Trigger**: Connected from Main Menu Flow

### **Flow Steps**:

#### **Step 1: Generate Progress Report**
```
Widget: Run Function
Function Name: generateProgressReport
Parameters:
  - phoneNumber: {{flow.variables.phoneNumber}}
```

**Function Code**:
```javascript
exports.handler = function(context, event, callback) {
  const phoneNumber = event.phoneNumber;
  
  // Get user's progress data from database
  const progressData = {
    totalSessions: 15,
    currentStreak: 7,
    averageRating: 4.2,
    totalCalories: 2250,
    favoriteWorkout: "HIIT",
    weeklyGoal: 5,
    weeklyCompleted: 4,
    monthlyGoal: 20,
    monthlyCompleted: 15
  };
  
  // Calculate percentages
  const weeklyProgress = (progressData.weeklyCompleted / progressData.weeklyGoal) * 100;
  const monthlyProgress = (progressData.monthlyCompleted / progressData.monthlyGoal) * 100;
  
  callback(null, {
    progress: progressData,
    weeklyProgress: weeklyProgress,
    monthlyProgress: monthlyProgress
  });
};
```

#### **Step 2: Send Progress Report**
```
Widget: Send Message
To: {{flow.variables.phoneNumber}}
Body: |
  📊 Your Fitness Progress Report
  
  🎯 This Week: {{widgets.generate_report.parsed.progress.weeklyCompleted}}/{{widgets.generate_report.parsed.progress.weeklyGoal}} workouts
  📈 Weekly Progress: {{widgets.generate_report.parsed.weeklyProgress}}%
  
  🎯 This Month: {{widgets.generate_report.parsed.progress.monthlyCompleted}}/{{widgets.generate_report.parsed.progress.monthlyGoal}} workouts
  📈 Monthly Progress: {{widgets.generate_report.parsed.monthlyProgress}}%
  
  🔥 Total Calories Burned: {{widgets.generate_report.parsed.progress.totalCalories}}
  ⭐ Average Rating: {{widgets.generate_report.parsed.progress.averageRating}}/5
  🔥 Current Streak: {{widgets.generate_report.parsed.progress.currentStreak}} days
  💪 Favorite Workout: {{widgets.generate_report.parsed.progress.favoriteWorkout}}
  
  {{#if (gt widgets.generate_report.parsed.weeklyProgress 80)}}
  🎉 You're crushing your goals! Keep it up!
  {{else}}
  💪 You're making great progress! Let's push for more!
  {{/if}}
  
  Reply "Workout" for today's session or "Help" for more options.
```

---

## 📅 Flow 5: Meeting Flow

### **Purpose**: Schedule 1-on-1 coaching sessions

### **Trigger**: Connected from Main Menu Flow

### **Flow Steps**:

#### **Step 1: Send Meeting Options**
```
Widget: Send Message
To: {{flow.variables.phoneNumber}}
Body: |
  📅 Schedule a 1-on-1 Coaching Session
  
  I'd love to help you reach your fitness goals! Here are available time slots:
  
  🕐 Today:
  • 2:00 PM - 30 min session
  • 4:00 PM - 30 min session
  • 6:00 PM - 30 min session
  
  🕐 Tomorrow:
  • 10:00 AM - 30 min session
  • 2:00 PM - 30 min session
  • 5:00 PM - 30 min session
  
  Reply with your preferred time (e.g., "Today 2 PM" or "Tomorrow 10 AM")
  
  Or reply "Cancel" to go back to main menu.
```

#### **Step 2: Wait for Time Selection**
```
Widget: Wait for User Input
Timeout: 3600 (1 hour)
```

#### **Step 3: Parse Time Selection**
```
Widget: Run Function
Function Name: parseMeetingTime
Parameters:
  - message: {{widgets.wait_for_time.parsed.Body}}
  - phoneNumber: {{flow.variables.phoneNumber}}
```

**Function Code**:
```javascript
exports.handler = function(context, event, callback) {
  const message = event.message.toLowerCase();
  const phoneNumber = event.phoneNumber;
  
  let meetingTime = null;
  let meetingDate = null;
  
  if (message.includes('today') && message.includes('2')) {
    meetingTime = '14:00';
    meetingDate = new Date().toISOString().split('T')[0];
  } else if (message.includes('today') && message.includes('4')) {
    meetingTime = '16:00';
    meetingDate = new Date().toISOString().split('T')[0];
  } else if (message.includes('today') && message.includes('6')) {
    meetingTime = '18:00';
    meetingDate = new Date().toISOString().split('T')[0];
  } else if (message.includes('tomorrow') && message.includes('10')) {
    meetingTime = '10:00';
    const tomorrow = new Date();
    tomorrow.setDate(tomorrow.getDate() + 1);
    meetingDate = tomorrow.toISOString().split('T')[0];
  }
  // Add more time parsing logic...
  
  callback(null, {
    meetingTime: meetingTime,
    meetingDate: meetingDate,
    isValid: !!meetingTime
  });
};
```

#### **Step 4: Confirm Meeting**
```
Widget: Send Message
To: {{flow.variables.phoneNumber}}
Body: |
  ✅ Meeting Confirmed!
  
  📅 Date: {{widgets.parse_time.parsed.meetingDate}}
  🕐 Time: {{widgets.parse_time.parsed.meetingTime}}
  ⏱️ Duration: 30 minutes
  📞 Platform: WhatsApp Video Call
  
  I'll send you a reminder 15 minutes before our session.
  
  In the meantime, keep up with your daily workouts! 💪
  
  Reply "Cancel" if you need to reschedule.
```

---

## 🤖 Flow 6: AI Chat Flow

### **Purpose**: Provide intelligent responses to user questions

### **Trigger**: Connected from Main Menu Flow

### **Flow Steps**:

#### **Step 1: Get AI Response**
```
Widget: Run Function
Function Name: aiResponse
Parameters:
  - message: {{flow.variables.originalMessage}}
  - phoneNumber: {{flow.variables.phoneNumber}}
  - userProfile: {{flow.variables.userProfile}}
```

**Function Code**:
```javascript
exports.handler = function(context, event, callback) {
  const OpenAI = require('openai');
  const openai = new OpenAI({
    apiKey: context.OPENAI_API_KEY
  });
  
  const userMessage = event.message;
  const userProfile = JSON.parse(event.userProfile || '{}');
  
  const prompt = `You are an AI fitness coach. User profile: ${JSON.stringify(userProfile)}
  
  User message: ${userMessage}
  
  Respond as a supportive, knowledgeable fitness coach. Keep responses under 200 words. Be encouraging and helpful.`;
  
  openai.chat.completions.create({
    model: "gpt-3.5-turbo",
    messages: [{ role: "user", content: prompt }],
    max_tokens: 300
  })
  .then(response => {
    callback(null, { 
      success: true, 
      response: response.choices[0].message.content 
    });
  })
  .catch(error => {
    callback(null, { 
      success: false, 
      response: "I'm here to help with your fitness journey! What would you like to know about workouts, nutrition, or motivation?" 
    });
  });
};
```

#### **Step 2: Send AI Response**
```
Widget: Send Message
To: {{flow.variables.phoneNumber}}
Body: |
  {{widgets.get_ai_response.parsed.response}}
  
  💡 Need more help? Reply with:
  • "Workout" - Get today's workout
  • "Progress" - Check your stats
  • "Schedule" - Book a session
  • "Help" - See all options
```

---

## ⏰ Flow 7: Scheduled Workout Delivery

### **Purpose**: Automatically send daily workout videos

### **Trigger**: Schedule (Daily at 6 PM)

### **Flow Steps**:

#### **Step 1: Get Users for Workout Delivery**
```
Widget: Run Function
Function Name: getUsersForWorkout
Parameters:
  - currentTime: {{flow.variables.currentTime}}
```

**Function Code**:
```javascript
exports.handler = function(context, event, callback) {
  const currentTime = event.currentTime;
  
  // Get users who prefer this time slot
  const users = [
    { phoneNumber: 'whatsapp:+1234567890', name: 'John', preferredTime: '18:00' },
    { phoneNumber: 'whatsapp:+0987654321', name: 'Sarah', preferredTime: '18:00' }
  ];
  
  callback(null, {
    users: users
  });
};
```

#### **Step 2: Send Workout to Each User**
```
Widget: Send Message
To: {{widgets.get_users.parsed.users[0].phoneNumber}}
Body: |
  🏋️‍♀️ Time for your daily workout, {{widgets.get_users.parsed.users[0].name}}!
  
  Today's workout: Full Body HIIT
  ⏱️ Duration: 20 minutes
  🔥 Calories: 150-200
  
  🎥 Watch here: https://www.youtube.com/watch?v=dQw4w9WgXcQ
  
  Reply "Done" when you complete it or "Skip" if you need to skip today.
```

---

## 🔧 Advanced Flow Features

### **Error Handling**:
```
Widget: Split Based On
Variable: {{widgets.run_function.parsed.success}}
```

### **Retry Logic**:
```
Widget: Run Function
Widget: Split Based On (success)
Widget: Wait (if failed)
Widget: Run Function (retry)
```

### **User Input Validation**:
```
Widget: Run Function (validate input)
Widget: Split Based On (isValid)
Widget: Send Message (error if invalid)
```

---

## 📊 Flow Analytics

### **Track Flow Performance**:
- **Execution Count**: How many times each flow runs
- **Success Rate**: Percentage of successful executions
- **Error Rate**: Percentage of failed executions
- **Average Duration**: Time taken for each flow

### **Monitor User Engagement**:
- **Response Rate**: How many users respond to messages
- **Completion Rate**: How many users complete workouts
- **Drop-off Points**: Where users stop engaging

---

## 🚀 Deployment Checklist

### **Before Launch**:
- [ ] Test all flows with your phone number
- [ ] Verify Google Forms integration
- [ ] Test AI responses
- [ ] Check scheduled flows
- [ ] Set up monitoring alerts
- [ ] Configure error handling

### **After Launch**:
- [ ] Monitor flow execution logs
- [ ] Track user engagement metrics
- [ ] Optimize flows based on feedback
- [ ] Scale up as user base grows

---

**Your Twilio Studio fitness bot is now ready to transform your fitness coaching business! 🚀**
