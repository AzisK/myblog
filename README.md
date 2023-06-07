# myblog

## Development

### Serve
This command serves all the blog locally
```zsh
hugo serve
```

### Issues

A. It says that port 1313 is already in use and hangs

- Check for running `hugo`s `ps aux | grep hugo`
- Check if something else is using 1313 port `lsof -i :1313`
- Kill the port by typing `kill -9 <PORT_NUMBER>`
