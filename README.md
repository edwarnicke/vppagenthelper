[vpp-agent](https://github.com/ligato/vpp-agent) provides a convenient GRPC API for [vpp](https://fd.io/documentation/).

vppagenthelper is a simple tool to make it easy to run a (vpp-agent,vpp) pair from your go code.

```go
import (
   "github.com/edwarnicke/vppagenthelper"
   "go.ligato.io/vpp-agent/v3/proto/ligato/configurator"
)

ctx,cancel := context.WithCancel(context.Background())

// Runs an instance of vpp and vpp-agent, returning a grpc.ClientConnInterface (cc) connected to the started vpp-agent
// and an errCh that will receive any errors.
cc, errCh := vppagenthelper.StartAndDialContext(ctx)

// An error may already be in channel on launch, otherwise errCh
// will be closed when vpp and vpp-agent have both exited
select {
case err := <-errCh:
    logrus.Fatal(err)
default:
}

client := configurator.NewConfiguratorServiceClient(cc)

// Stop vpp and vpp-agent
cancel()
// Block till they have exited
<-errCh
```

# Options

Both vpp and vpp-agent have default notions of where various config files and socket files should live.
If you wish to start more than one instance of vpp-agent, you will need to place them under an alternate rootDir.
This can be done with vppagenthelper.Options

```go
import (
    "github.com/phayes/freeport"
    "github.com/edwarnicke/vppagenthelper"
)

rootDir, err = ioutil.TempDir("", "vppagent-")
grpcPort, err := freeport.GetFreePort()
httpPort, err := freeport.GetFreePort()
f.Require().NoError(err)
cc, errCh = vppagenthelper.StartAndDialContext(
    f.ctx,
    vppagenthelper.WithRootDir(rootDir),
    vppagenthelper.WithGrpcPort(grpcPort),
    vppagenthelper.WithHTTPPort(httpPort),
)
```

# Using vppctl with an alternate rootDir
To use vppctl when using an alternate rootDir run:

```
vppctl -s ${ROOTDIR}/var/run/vpp/cli.sock ${normal cmds}
```

# Using agentctl with an alternate rootDir
To use agentctl when using an alternate rootDir run:

```
agentctl --http-port ${HTTP_PORT}
```



