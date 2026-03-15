# myblog

## Development

### A. Install

Install Hugo with

```zsh
brew install hugo
```

### B. Develop

```zsh
# Create a new post (uses archetype template)
# The path is relative to content/ directory
hugo new post/your-post-title.md

# For English posts, add language suffix:
hugo new post/your-post-title.en.md
```

The archetype template at `archetypes/default.md` creates a new post with a defined frontmatter.

The post will have `draft: true` by default in this case, so it won't be visible until you remove it.

```zsh
# Build static site to /public/
hugo

# Serve all the blog locally
hugo serve

# Include draft posts
hugo serve -D

# Check Hugo version
hugo version
```

### C. Images

When adding images, use relative paths from the post's directory. To link to files in `static/`, use `..` to go up one level:

```markdown
![alt](../image.jpg)
```

### D. Issues

DA. It says that port 1313 is already in use and hangs

- Check for running `hugo`s `ps aux | grep hugo`
- Check if something else is using 1313 port `lsof -i :1313`
- Kill the port by typing `kill -9 <PORT_NUMBER>`
