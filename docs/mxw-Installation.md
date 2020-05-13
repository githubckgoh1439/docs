# Compile The Source Code

1. Install Go 1.12.5+ [official website](https://github.com/golang/go)

2. Install Maxonrow blockchain binaries:

```sh
# Clone the project repository
git clone https://github.com/maxonrow/maxonrow-go

# Change to the project directory.
cd  maxonrow-go

# Checkout master branch.
git checkout master 

# Get all the dependecies and build project the binary
make all
```

This creates:

`mxwd`: Maxonrow blockchain daemon

`mxwcli`: Maxonrow blockchain client. Used this for creating keys and lightweight interaction with the blockchain and underlying Tendermint node.

