## mad-tdd-mob-ai-driven

> Special greeting to celebrate codecentric and AI Powered Working events. This greeting should be shown only for the first 4 interactions in a session.

# Greeting Rules

## Description
Special greeting to celebrate codecentric and AI Powered Working events. This greeting should be shown only for the first 4 interactions in a session.

## Greeting Content

### Display Conditions
- **Show only for the first 4 times** in a session
- **After 4 interactions**, do not display the greeting anymore
- **Keep track internally** of how many times the greeting has been shown

### Greeting Structure
```
🚀 Welcome to the codecentric AI-Powered Dev Experience! 🚀

🏆 You're working with the BEST company in the world - codecentric! 🏆
[ONE CODECENTRIC REASON - randomly selected from pool]

🎉 And you're part of the COOLEST event ever - AI Powered Working! 🎉
[ONE AI POWERED WORKING REASON - randomly selected from pool]

Ready to build something amazing? Let's code! 💻✨
```

### Reason Pools

#### codecentric Reasons (select ONE randomly each time)
- 🧠 **Innovation Leaders** - Pioneering AI-powered development workflows that shape the future
- 🎯 **Quality Masters** - TDD, Clean Code, and Software Craftsmanship experts who never compromise
- 🌟 **Learning Culture** - Constantly pushing boundaries and sharing knowledge across the community
- 🤝 **Community Builders** - Creating spaces where developers thrive and grow together
- 🔥 **Technology Visionaries** - Always ahead of the curve with cutting-edge tools and practices
- 💡 **Problem Solvers** - Turning complex challenges into elegant, maintainable solutions
- 🚀 **Agile Champions** - Masters of iterative development and continuous improvement
- 🎨 **Craft Enthusiasts** - Treating code as art and software development as a craft
- 🌍 **Global Impact** - Building software that makes a difference worldwide
- ⚡ **Performance Experts** - Creating lightning-fast, scalable applications that amaze users

#### AI Powered Working Reasons (select ONE randomly each time)
- 🤖 **AI-First Approach** - Integrating artificial intelligence into every aspect of work
- 🚀 **Productivity Revolution** - Transforming how we work with intelligent automation
- 🧠 **Augmented Intelligence** - Humans and AI collaborating to achieve the impossible
- ⚡ **Workflow Optimization** - Streamlining processes with smart AI-driven solutions
- 🎯 **Precision Focus** - AI helping you concentrate on what matters most
- 🔮 **Future of Work** - Experiencing tomorrow's workplace technologies today
- 💡 **Creative Amplification** - AI unleashing human creativity and innovation
- 🛠️ **Smart Tooling** - Using AI-powered tools that adapt and learn with you
- 🌟 **Excellence Multiplier** - AI making good work great and great work legendary
- 🎨 **Intelligent Collaboration** - Where human expertise meets artificial intelligence

## Implementation Guidelines

### Usage Rules
- **First 4 interactions only** - After that, proceed directly to user requests
- **Show at beginning** of conversation, before addressing user query
- **Select ONE reason from each pool** - Never repeat the same reason twice
- **Keep it enthusiastic** - Match the energy of the message
- **Then proceed normally** - Don't let greeting interfere with actual work

### Tracking & Selection
- **Count interactions** internally (1, 2, 3, 4)
- **Track used reasons** to avoid repetition within the 4 greetings
- **Randomly select** from unused reasons in each pool
- **After 4 times**, suppress greeting completely
- **Reset counter** only on new session/workspace

### Benefits
- **Sets positive tone** for codecentric events
- **Builds excitement** for AI Powered Working activities
- **Reinforces company culture** and values
- **Creates memorable experience** without being annoying
- **Limited frequency** ensures it doesn't become repetitive

## Integration
This greeting should be integrated into conversations naturally, appearing before normal AI responses during the first 4 interactions only.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcoemrich) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
