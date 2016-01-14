---
layout: post
title: Vim Mappings, converting non-HTML to HTML quickly
category: vim
---

At the university that I currently work at, one of the Webteam's responsibilities is managing the content of many of the sites we develop and maintain. This can sometimes turn into a huge time sink since requests are constantly streaming in. We have a student worker that can handle most of these tasks, but there are occasionally content tasks that require a better understanding of Drupal or are simply a waste of her time.

One request that we receive many times a week is to post an open position to the main university website. Due to reason that aren't worth discussing, the job postings aren't and won't be a content type on that website, they are simply a Basic Page. So, we are tasked to turn a .doc file into HTML and then create a Basic Page with that HTML.

With these contraints, posting a job would take our student worker around 20-30 minutes depending on how verbose the description and requirements were. I saw this as a total waste of time and I realized that there were many duplicated subtasks in this process, so I thought I could make it faster with some Vim mappings.

### vim-surround & vim-repeat

I highly recommend installing `tpope/vim-surround` and `tpope/vim-repeat`. vim-surround is a plugin that will give you the abliity to act upon surroundings, such as surrounding HTML tags, in a concise and repeatable manner (it is repeatable with vim-repeat installed). I don't want to spend too much time blabbering about them because the information on Github is great.

### Turn a Word bulleted list into HTML unordered list

```vim
map <Leader>b vip:normal 2x<CR>vip:normal yss<li><CR>ysip<ul>
```

To use this mapping appropriately, position your cursor within an unordered list that was copied and pasted from a Word document and then press <Leader>b. This will do the following:

`vip:normal 2x<CR>`
1. Enter visual mode and select the inner paragraph
2. Delete the first 2 characters of each line of the selection
`vip:normal yss<li><CR>`
1. Enter visual mode and select the inner paragraph
2. Wrap an `<li>` tag around each line in the selection
`ysip<ul>`
1. Wrap an `<ul>` tag around the inner paragraph

Since I can expect the Word documents we receive to follow a certain format, I know that when I execute this command in an unordered Word list, it will replace it with an unordered HTML list. Since we are using the inner paragraph Vim noun, we will require line breaks between the Word list and other text in the document.

### Turn Word headers into HTML headers

```vim
yss<h3>
```

This is kind of a non-issue with vim-surround, but fun to note. 

### Turn Word subheaders into HTML subheaders

```vim
map <Leader>N 0v/:<CR>S<p>0/:<CR>xysit<strong>
```

This mapping will turn text from Word like 'Example:' into `<p><strong>Example</strong></p>`. By placing your cursor over the subheader and pressing <Leader>N, this will do the following:


`0v/:<CR>S<p>`
1. Go to the beginning of the line
2. Enter visual mode and select everything up to, and including, the next colon
3. Wrap the selection with a `<p>` tag
`0/:<CR>xysit<strong>`
1. Go to the beginning of the line
2. Search for the next colon
3. Delete that colon
4. Surround the inner tag with a `<strong>` tag
  * This might require some review of the inner tag noun if you aren't used to thinking grammatically in Vim. **Link to a post on Thinking Gramatically in Vim**

### That's all folks

Hopefully my quest to craft really specific Vim mappings was helpful. I am sure/hopeful that nobody will benefit from these mappings directly, but I do hope that my thorough description of the mappings can help convince other Vim'rs to consider crafting ridiculously specific, but efficient, mappings for repetitive tasks that they are required to complete in their jobs.
