## ckd

> Streamlit chat interfaces don't automatically scroll to show new content when messages are added. Users have to manually scroll to see new questions and responses, creating poor UX.

# Chat Auto-Scroll Implementation

## Problem
Streamlit chat interfaces don't automatically scroll to show new content when messages are added. Users have to manually scroll to see new questions and responses, creating poor UX.

## Solution in [main.py](mdc:main.py)

### Auto-Scroll Helper Function
```python
def scroll_to_bottom():
    """Add JavaScript to auto-scroll chat to bottom"""
    st.markdown(
        """
        <script>
            function scrollToBottom() {
                // Scroll the main content area smoothly
                parent.window.scrollTo({
                    top: parent.document.body.scrollHeight,
                    behavior: 'smooth'
                });
                
                // Also try to scroll any chat containers
                var containers = parent.document.querySelectorAll('[data-testid="stVerticalBlock"]');
                containers.forEach(function(container) {
                    container.scrollTop = container.scrollHeight;
                });
            }
            // Delay scroll to ensure content is rendered
            setTimeout(scrollToBottom, 150);
        </script>
        """,
        unsafe_allow_html=True
    )
```

### Usage Pattern
Call `scroll_to_bottom()` after every message addition:

```python
# After adding any message to chat history
st.session_state.messages.append({"role": "assistant", "content": question})
scroll_to_bottom()  # Auto-scroll to show new content

# After displaying user responses
with st.chat_message("user"):
    st.markdown(user_response)
scroll_to_bottom()  # Auto-scroll to show user message

# After displaying chat history
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])
if len(st.session_state.messages) > 0:
    scroll_to_bottom()  # Ensure visibility of latest messages
```

## Implementation Points

### 1. Chat Container Setup
```python
# Create chat container for better organization
chat_container = st.container()

with chat_container:
    for message in st.session_state.messages:
        with st.chat_message(message["role"]):
            st.markdown(message["content"])
```

### 2. Auto-Scroll Triggers
Auto-scroll should be triggered after:
- ✅ Initial assistant greeting
- ✅ Each new question display
- ✅ Each user response
- ✅ Final assessment results
- ✅ Any new message addition

### 3. JavaScript Strategy
The solution uses dual approach:
- **Smooth scrolling** with `behavior: 'smooth'` for better UX
- **Multiple selectors** to handle different Streamlit container types
- **Delayed execution** (150ms) to ensure content is fully rendered
- **Parent window access** to work within Streamlit's iframe structure

## Browser Compatibility
- ✅ Chrome/Edge: Full support for smooth scrolling
- ✅ Firefox: Full support
- ⚠️ Safari: Basic support (may not be as smooth)
- ✅ Mobile browsers: Generally supported

## Alternative Approaches

### CSS-Only (Limited)
```css
/* Add to custom CSS - limited effectiveness */
.main {
    scroll-behavior: smooth;
}
```

### Streamlit Native (Future)
```python
# When available in future Streamlit versions
st.chat_container(auto_scroll=True)
```

## Troubleshooting

### If Auto-Scroll Doesn't Work:
1. **Check JavaScript Console**: Look for errors in browser dev tools
2. **Increase Delay**: Change timeout from 150ms to 300ms
3. **Disable Browser Extensions**: Some ad blockers may interfere
4. **Test Different Browsers**: Ensure compatibility

### Common Issues:
- **Too Fast**: Scroll happens before content renders → Increase delay
- **Iframe Issues**: Streamlit runs in iframe → Use `parent.window`
- **Container Detection**: Multiple containers → Target all possible selectors

## Best Practices
1. **Always call after message changes** - Don't forget any interaction point
2. **Use consistent delay timing** - 150ms works for most cases
3. **Test on different devices** - Mobile behavior may vary
4. **Provide manual scroll option** - Some users prefer manual control
5. **Log scroll attempts** - Help with debugging if issues arise

This implementation ensures smooth, automatic scrolling in the CKD assessment chat interface, improving user experience during the question/answer flow.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/prashiyn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
