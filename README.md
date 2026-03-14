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
hugo new post/your-post-title.md

# Edit the post in your editor of choice
# Then continue with build/serve commands below
```

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

### C. Issues

CA. It says that port 1313 is already in use and hangs

- Check for running `hugo`s `ps aux | grep hugo`
- Check if something else is using 1313 port `lsof -i :1313`
- Kill the port by typing `kill -9 <PORT_NUMBER>`
