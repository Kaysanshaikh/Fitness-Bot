# 🚀 Twilio Studio WhatsApp Fitness Bot Setup Guide

Complete guide to build your AI-powered fitness coaching bot using Twilio Studio - the most cost-effective solution for your fitness business.

## 📋 Table of Contents
1. [Prerequisites & Account Setup](#prerequisites--account-setup)
2. [WhatsApp Business API Setup](#whatsapp-business-api-setup)
3. [Twilio Studio Flow Creation](#twilio-studio-flow-creation)
4. [Google Forms Integration](#google-forms-integration)
5. [AI Integration](#ai-integration)
6. [Testing & Deployment](#testing--deployment)
7. [Cost Optimization](#cost-optimization)
8. [Troubleshooting](#troubleshooting)

---

## 🔧 Prerequisites & Account Setup

### Step 1: Create Twilio Account
1. **Visit**: https://www.twilio.com/
2. **Sign Up**: Use your email and create a password
3. **Verify Email**: Check your email and click verification link
4. **Complete Profile**: Add your business information

### Step 2: Get Free Trial Credits
- **Free Trial**: $15-20 in free credits
- **Duration**: Valid for 30 days
- **Usage**: Perfect for testing your fitness bot

### Step 3: Install Required Tools
```bash
# Install Twilio CLI (optional but helpful)
npm install -g twilio-cli

# Or download from: https://www.twilio.com/docs/twilio-cli
```

---

## 📱 WhatsApp Business API Setup

### Step 1: Enable WhatsApp in Twilio Console
1. **Login** to Twilio Console
2. **Navigate** to Messaging → Try it out → Send a WhatsApp message
3. **Follow** the setup wizard
4. **Note** your WhatsApp number: `whatsapp:+14155238886`

### Step 2: WhatsApp Sandbox Setup
1. **Go to**: Console → Messaging → Try it out → Send a WhatsApp message
2. **Join Sandbox**: Send the provided code to the WhatsApp number
3. **Test Message**: Send a test message to verify setup

### Step 3: Business API Approval (Optional)
For production use, apply for WhatsApp Business API:
1. **Contact** Twilio support
2. **Provide** business documentation
3. **Wait** for approval (2-4 weeks)
4. **Get** your business WhatsApp number

---

## 🎨 Twilio Studio Flow Creation

### Step 1: Create New Studio Flow
1. **Navigate** to Twilio Console → Studio
2. **Click** "Create new Flow"
3. **Name**: "Fitness Bot - Main Flow"
4. **Description**: "AI-powered fitness coaching bot"
5. **Click** "Create"

### Step 2: Configure Trigger
1. **Select** "Webhook" trigger
2. **Configure**:
   - **Trigger Type**: Incoming Message
   - **Channel**: WhatsApp
   - **Event Type**: onMessageReceived

### Step 3: Build the Main Flow

#### **Flow Structure Overview:**
```
Webhook Trigger
    ↓
Check User Status
    ↓
[New User] → Welcome Flow
    ↓
[Existing User] → Main Menu
    ↓
Route to Functions
    ↓
Send Response
```

#### **Detailed Flow Configuration:**

**1. Welcome Flow (New Users)**
```
Trigger: onMessageReceived
    ↓
Run Function: checkUserStatus
    ↓
Split Based On: userExists
    ↓
[No] → Send Welcome Message
    ↓
Run Function: createUserProfile
    ↓
Send: "Welcome! I'm your AI fitness coach..."
```

**2. Main Menu (Existing Users)**
```
Split Based On: userExists
    ↓
[Yes] → Run Function: parseUserIntent
    ↓
Split Based On: intent
    ↓
[workout_request] → Send Workout Video
    ↓
[progress_check] → Send Progress Report
    ↓
[nutrition_advice] → Send Nutrition Tips
    ↓
[schedule_meeting] → Meeting Scheduler
    ↓
[general_question] → AI Response
```

---

## 🔗 Google Forms Integration

### Step 1: Create Google Form
Create a form with these fields:
- **Full Name** (Text)
- **WhatsApp Number** (Text)
- **Age** (Number)
- **Fitness Goals** (Multiple Choice)
- **Preferred Time** (Multiple Choice)
- **Experience Level** (Multiple Choice)
- **Health Conditions** (Text, optional)

### Step 2: Google Apps Script Setup
1. **Open** your Google Form
2. **Click** Responses → More → Script editor
3. **Replace** code with:

```javascript
function onFormSubmit(e) {
  const response = e.response;
  const itemResponses = response.getItemResponses();
  
  // Extract form data
  const userData = {
    name: itemResponses[0].getResponse(),
    phone: formatPhoneNumber(itemResponses[1].getResponse()),
    age: parseInt(itemResponses[2].getResponse()),
    goals: itemResponses[3].getResponse(),
    preferredTime: itemResponses[4].getResponse(),
    experience: itemResponses[5].getResponse(),
    healthConditions: itemResponses[6] ? itemResponses[6].getResponse() : ''
  };
  
  // Send to Twilio Studio webhook
  const webhookUrl = 'https://webhooks.twilio.com/v1/...'; // Your Studio webhook URL
  
  UrlFetchApp.fetch(webhookUrl, {
    method: 'POST',
    contentType: 'application/json',
    payload: JSON.stringify({
      userData: userData,
      source: 'google_form',
      timestamp: new Date().toISOString()
    })
  });
}

function formatPhoneNumber(phone) {
  // Remove all non-digits
  let cleaned = phone.replace(/\D/g, '');
  
  // Add country code if missing
  if (cleaned.length === 10) {
    cleaned = '91' + cleaned; // Assuming India (+91)
  }
  
  return 'whatsapp:+' + cleaned;
}
```

### Step 3: Connect to Twilio Studio
1. **Get** your Studio webhook URL
2. **Update** the webhook URL in Google Apps Script
3. **Test** form submission

---

## 🤖 AI Integration

### Step 1: OpenAI Integration Function
Create a Twilio Function for AI responses:

```javascript
// Function: aiResponse
exports.handler = function(context, event, callback) {
  const OpenAI = require('openai');
  const openai = new OpenAI({
    apiKey: context.OPENAI_API_KEY
  });
  
  const userMessage = event.Body;
  const userProfile = event.userProfile || {};
  
  const prompt = `You are an AI fitness coach. User profile: ${JSON.stringify(userProfile)}
  
  User message: ${userMessage}
  
  Respond as a supportive, knowledgeable fitness coach. Keep responses under 200 words.`;
  
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
      response: "I'm here to help with your fitness journey! What would you like to know?" 
    });
  });
};
```

### Step 2: Configure AI Function
1. **Go to** Twilio Console → Functions
2. **Create** new function: "aiResponse"
3. **Paste** the code above
4. **Add** environment variable: `OPENAI_API_KEY`
5. **Deploy** the function

### Step 3: Connect AI to Studio Flow
1. **Add** "Run Function" widget to your flow
2. **Configure**:
   - **Function**: aiResponse
   - **Parameters**: 
     - Body: `{{trigger.message.Body}}`
     - userProfile: `{{flow.variables.userProfile}}`
3. **Connect** to "Send Message" widget

---

## 🎯 Complete Flow Examples

### **Flow 1: Daily Workout Delivery**
```
Trigger: Schedule (Daily at 6 PM)
    ↓
Run Function: getDailyWorkout
    ↓
Split Based On: user.preferredTime
    ↓
[6 PM] → Send Workout Video
    ↓
Wait: 2 hours
    ↓
Send: "How was your workout today?"
    ↓
Wait: 1 hour
    ↓
Run Function: sendMotivation
```

### **Flow 2: Progress Tracking**
```
Trigger: onMessageReceived
    ↓
Run Function: parseIntent
    ↓
Split Based On: intent
    ↓
[progress_check] → Run Function: generateProgressReport
    ↓
Send: Progress Report Message
    ↓
Run Function: updateAnalytics
```

### **Flow 3: Meeting Scheduler**
```
Trigger: onMessageReceived
    ↓
Run Function: parseIntent
    ↓
Split Based On: intent
    ↓
[schedule_meeting] → Send: "When would you like to schedule?"
    ↓
Wait: User Response
    ↓
Run Function: parseDateTime
    ↓
Send: "Meeting scheduled for [date/time]"
    ↓
Run Function: createCalendarEvent
```

---

## 🧪 Testing & Deployment

### Step 1: Test Individual Components
1. **Test** WhatsApp connection
2. **Test** Google Forms integration
3. **Test** AI responses
4. **Test** scheduled flows

### Step 2: End-to-End Testing
1. **Submit** test form data
2. **Verify** welcome message received
3. **Test** workout request
4. **Test** progress check
5. **Test** AI conversation

### Step 3: Production Deployment
1. **Review** all flows
2. **Set** production webhook URLs
3. **Configure** monitoring
4. **Launch** to users

---

## 💰 Cost Optimization

### **Monthly Cost Breakdown:**
- **100 users**: $15-30/month
- **500 users**: $50-100/month
- **1000 users**: $100-200/month

### **Cost-Saving Tips:**
1. **Use** Twilio's free trial credits
2. **Optimize** message frequency
3. **Batch** messages when possible
4. **Monitor** usage in Twilio Console
5. **Set** usage alerts

### **Pricing Structure:**
- **WhatsApp Messages**: $0.0075 per message
- **Studio Flows**: Free
- **Functions**: $0.0001 per execution
- **Phone Numbers**: $1/month per number

---

## 🔧 Troubleshooting

### **Common Issues:**

#### **1. WhatsApp Messages Not Sending**
- **Check**: Sandbox activation
- **Verify**: Phone number format
- **Test**: With Twilio's test number

#### **2. Google Forms Not Triggering**
- **Check**: Apps Script permissions
- **Verify**: Webhook URL
- **Test**: Manual script execution

#### **3. AI Responses Not Working**
- **Check**: OpenAI API key
- **Verify**: Function deployment
- **Test**: Function directly

#### **4. Flow Not Executing**
- **Check**: Trigger configuration
- **Verify**: Webhook URL accessibility
- **Test**: Flow execution logs

### **Debug Steps:**
1. **Check** Twilio Console logs
2. **Verify** environment variables
3. **Test** individual components
4. **Review** flow execution history

---

## 📊 Monitoring & Analytics

### **Twilio Console Monitoring:**
- **Flow Execution**: Track flow performance
- **Message Delivery**: Monitor delivery rates
- **Error Rates**: Identify issues quickly
- **Usage Analytics**: Track costs

### **Custom Analytics:**
- **User Engagement**: Track user interactions
- **Workout Completion**: Monitor fitness progress
- **AI Usage**: Track AI response effectiveness
- **Cost Analysis**: Monitor spending

---

## 🚀 Next Steps

### **Immediate Actions:**
1. **Create** Twilio account
2. **Set up** WhatsApp sandbox
3. **Build** basic welcome flow
4. **Test** with your phone number

### **Week 1 Goals:**
- ✅ Complete basic flow setup
- ✅ Test Google Forms integration
- ✅ Implement AI responses
- ✅ Test with 5-10 users

### **Month 1 Goals:**
- ✅ Launch to 50+ users
- ✅ Optimize flows based on feedback
- ✅ Implement advanced features
- ✅ Monitor and optimize costs

---

## 📞 Support Resources

- **Twilio Documentation**: https://www.twilio.com/docs/studio
- **WhatsApp Business API**: https://www.twilio.com/docs/whatsapp
- **Twilio Support**: Available in Console
- **Community Forum**: https://www.twilio.com/community

---

**Ready to build your fitness bot? Start with Step 1 and you'll have a working AI-powered fitness coach in just a few hours! 🚀**
