### 🌱 **Phase 1: Foundation Setup**
1. **Chrome Extension Basics**
   - Create `manifest.json` (MV3) with:
     ```json
     {
       "manifest_version": 3,
       "permissions": ["activeTab", "scripting", "storage"],
       "host_permissions": ["*://*.reddit.com/*"],
       "content_scripts": [{
         "matches": ["*://*.reddit.com/*"],
         "js": ["contentScript.js"]
       }]
     }
     ```
   - Add icons (16px, 48px, 128px)

2. **Content Script Injection**
   - Create `contentScript.js` to:
     - Detect Reddit post pages (`window.location.href` check)
     - Inject floating "ScrollLess" button into Reddit's DOM
     ```javascript
     const button = document.createElement('button');
     button.id = 'scrollless-trigger';
     button.textContent = 'ScrollLess';
     // Position near Reddit's post title
     document.querySelector('[data-test-id="post-content"]').append(button);
     ```

---

### 🛠️ **Phase 2: Data Pipeline**
3. **Reddit Data Scraping**
   - For posts:
     ```javascript
     const post = {
       title: document.querySelector('[data-adclicklocation="title"]').innerText,
       text: document.querySelector('[data-test-id="post-content"]').innerText,
       author: document.querySelector('[data-testid="post_author_link"]').innerText,
       upvotes: document.querySelector('[data-test-id="post-upvote-button"]').innerText
     };
     ```
   - For comments (limit to top 50):
     ```javascript
     const comments = Array.from(document.querySelectorAll('[data-testid="comment"]'))
       .slice(0, 50).map(comment => ({
         text: comment.querySelector('[data-testid="comment"]').innerText,
         upvotes: comment.querySelector('[id^="vote-arrows"]').innerText,
         replies: comment.querySelectorAll('[data-testid="comment"]').length
       }));
     ```

4. **Data Storage & Processing**
   - Use `chrome.storage.local` to cache raw data
   - Create data sanitizer:
     ```javascript
     const cleanText = (text) => text.replace(/[\r\n\t]/g, ' ').trim();
     ```

---

### 🧠 **Phase 3: AI Processing Core**
5. **Summarization Engine (OpenAI API)**
   - Create API call template:
     ```javascript
     const getSummary = async (text) => {
       const prompt = `Summarize this Reddit post in 3 sentences with witty tone:\n\n${text}`;
       
       const response = await fetch('https://api.openai.com/v1/chat/completions', {
         method: 'POST',
         headers: {
           'Authorization': `Bearer ${API_KEY}`,
           'Content-Type': 'application/json'
         },
         body: JSON.stringify({
           model: "gpt-3.5-turbo",
           messages: [{role: "user", content: prompt}],
           temperature: 0.7
         })
       });
       
       return response.choices[0].message.content;
     };
     ```

6. **Specialized Prompts**
   - Hottest Take prompt:
     ```
     "Identify the most extreme opinion from these comments. Respond in max 2 sentences with emoji. Example format: '🔥 HOT TAKE: [summary]'"
     ```
   - Vibe Check prompt:
     ```
     "Analyze the emotional tone of these comments. Choose from: 🤗 Wholesome, 💀 Chaotic, ☢️ Toxic, 😎 Chill. Include 1-sentence explanation."
     ```

---

### 🖥️ **Phase 4: UI Implementation**
7. **Summary Modal Component**
   - Create React component (or vanilla JS):
     ```jsx
     function SummaryModal({ postSummary, commentsSummary }) {
       return (
         <div className="scrollless-modal">
           <div className="tabs">
             <button onClick={() => setTab('post')}>Post TL;DR</button>
             <button onClick={() => setTab('comments')}>Comment Digest</button>
           </div>
           {tab === 'post' && <p>{postSummary}</p>}
           {tab === 'comments' && (
             <div>
               <h3>🔥 Hottest Take</h3>
               <p>{hottestTake}</p>
               <h3>🌡️ Vibe Check</h3>
               <p>{vibeCheck}</p>
             </div>
           )}
         </div>
       );
     }
     ```
   - Style with Tailwind:
     ```css
     .scrollless-modal {
       @apply bg-white p-4 rounded-lg shadow-xl min-w-[300px];
       position: fixed;
       top: 20%;
       right: 20px;
       z-index: 9999;
     }
     ```

---

### 🧪 **Phase 5: Testing & Optimization**
8. **Reddit Compatibility Testing**
   - Test on:
     - Old Reddit (new.reddit.com)
     - Mobile web view
     - Different post types (text, image, video)
   - Handle errors:
     ```javascript
     try {
       // scraping logic
     } catch (error) {
       console.error('ScrollLess scraping error:', error);
       showErrorMessage('⚠️ Couldn\'t read this post - try another one!');
     }
     ```

9. **Performance Tweaks**
   - Cache summaries using `chrome.storage.local`
   - Add loading spinner during API calls
   - Implement rate limiting for free-tier OpenAI

---

### 🚀 **Phase 6: Advanced Features**
10. **Nuclear Comment Detection**
    ```javascript
    function findNuclearComment(comments) {
      return comments.reduce((mostControversial, comment) => {
        const ratio = comment.upvotes / (comment.replies + 1);
        return ratio < mostControversial.ratio ? 
          { ...comment, ratio } : mostControversial;
      }, { ratio: Infinity });
    }
    ```

11. **Shareable Summaries**
    - Generate encoded URL hashes for sharing
    - Add "Copy Summary" button with format options:
      ```
      📝 ScrollLess Summary
      Post TL;DR: [summary]
      Hottest Take: [take]
      Full post: [URL]
      ```

---

### 📦 **Phase 7: Deployment Prep**
12. **Privacy Compliance**
    - Add privacy policy page
    - Implement anonymous usage (no user tracking)

13. **Store Listing Assets**
    - Create promo video (screen record of workflow)
    - Design 1280x800 banner with "Scroll less, know more" tagline

---

### ⏭️ **Next Immediate Steps**
1. Clone starter template:
   ```bash
   git clone https://github.com/awesome-chrome-extension-starters/react-tailwind-extension
   ```
2. Implement Phase 1 (manifest + button injection)
3. Test basic Reddit DOM scraping

