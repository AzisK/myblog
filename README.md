# myblog

## Development

### A. Install

Install Hugo with 

```zsh
brew install hugo
```

### B. Serve

This command serves all the blog locally
```zsh
hugo serve
```

### C. Additional info

Check Hugo version with
```zsh
hugo version
```

### D. Issues

DA. It says that port 1313 is already in use and hangs

- Check for running `hugo`s `ps aux | grep hugo`
- Check if something else is using 1313 port `lsof -i :1313`
- Kill the port by typing `kill -9 <PORT_NUMBER>`
