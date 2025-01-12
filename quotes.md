---
layout: page
title: Quotes
permalink: /quotes/
---
<style>
.quote-container {
    max-width: 800px;
    margin: 0 auto;
}

.quote-box {
    padding: 20px;
    margin: 20px 0;
    border-radius: 5px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    transition: transform 0.3s ease;
}

.quote-box:hover {
    transform: translateY(-5px);
}

.quote-text {
    font-size: 1.2em;
    margin-bottom: 10px;
    font-style: italic;
    font-family: 'Georgia', serif;
}

.quote-author {
    font-weight: bold;
    text-align: right;
    font-family: 'Arial', sans-serif;
}

.bg-sage { background-color: #E4EFE7; }
.bg-lavender { background-color: #F0E6EF; }
.bg-peach { background-color: #FFE5D9; }

.copy-btn {
    background-color: #f0f0f0;
    border: none;
    padding: 5px 10px;
    border-radius: 3px;
    cursor: pointer;
    font-size: 0.8em;
    margin-top: 10px;
}

.copy-btn:hover {
    background-color: #e0e0e0;
}

@media (max-width: 600px) {
    .quote-box {
        margin: 10px 0;
    }
    .quote-text {
        font-size: 1em;
    }
}
</style>

<div class="quote-container">
    <!-- <h1>Favorite Quotes</h1> -->
    <p>A collection of based one-liners that resonate with me.</p>

    <div class="quote-box bg-sage">
        <div class="quote-text">"Ambition without action is fantasy."</div>
        <div class="quote-author">— Derek Sivers</div>
        <button class="copy-btn" onclick="copyToClipboard(this)">Copy</button>
    </div>

    <div class="quote-box bg-lavender">
        <div class="quote-text">"Pessimists sound smart. Optimists make money."</div>
        <div class="quote-author">— Nat Friedman</div>
        <button class="copy-btn" onclick="copyToClipboard(this)">Copy</button>
    </div>

    <div class="quote-box bg-peach">
        <div class="quote-text">"Humility is not thinking less of yourself, but thinking of yourself less"</div>
        <div class="quote-author">— C.S Lewis </div>
        <button class="copy-btn" onclick="copyToClipboard(this)">Copy</button>
    </div>
</div>

<script>
function copyToClipboard(btn) {
    const quoteBox = btn.closest('.quote-box');
    const quoteText = quoteBox.querySelector('.quote-text').innerText;
    const quoteAuthor = quoteBox.querySelector('.quote-author').innerText;
    const fullQuote = `${quoteText} ${quoteAuthor}`;

    navigator.clipboard.writeText(fullQuote).then(() => {
        btn.textContent = 'Copied!';
        setTimeout(() => {
            btn.textContent = 'Copy';
        }, 2000);
    });
}
</script>
