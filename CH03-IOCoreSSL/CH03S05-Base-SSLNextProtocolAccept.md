# 基础组件：SSLNextProtocolAccept

## 定义

```
class SSLNextProtocolAccept : public SessionAccept
{
public:
  SSLNextProtocolAccept(Continuation *, bool);
  ~SSLNextProtocolAccept();

  void accept(NetVConnection *, MIOBuffer *, IOBufferReader *);

  // Register handler as an endpoint for the specified protocol. Neither
  // handler nor protocol are copied, so the caller must guarantee their
  // lifetime is at least as long as that of the acceptor.
  bool registerEndpoint(const char *protocol, Continuation *handler);

  // Unregister the handler. Returns false if this protocol is not registered
  // or if it is not registered for the specified handler.
  bool unregisterEndpoint(const char *protocol, Continuation *handler);

  SLINK(SSLNextProtocolAccept, link);

private:
  int mainEvent(int event, void *netvc);
  SSLNextProtocolAccept(const SSLNextProtocolAccept &);            // disabled
  SSLNextProtocolAccept &operator=(const SSLNextProtocolAccept &); // disabled

  MIOBuffer *buffer; // XXX do we really need this?
  Continuation *endpoint;
  SSLNextProtocolSet protoset;
  bool transparent_passthrough;

  friend struct SSLNextProtocolTrampoline;
};

static SSLNetVConnection *
ssl_netvc_cast(int event, void *edata)
{
  union {
    VIO *vio;
    NetVConnection *vc;
  } ptr;

  switch (event) {
  case NET_EVENT_ACCEPT:
    ptr.vc = static_cast<NetVConnection *>(edata);
    return dynamic_cast<SSLNetVConnection *>(ptr.vc);
  case VC_EVENT_INACTIVITY_TIMEOUT:
  case VC_EVENT_READ_COMPLETE:
  case VC_EVENT_ERROR:
    ptr.vio = static_cast<VIO *>(edata);
    return dynamic_cast<SSLNetVConnection *>(ptr.vio->vc_server);
  default:
    return NULL;
  }
}
```

## 参考资料

- [P_SSLNextProtocolAccept.h](http://github.com/apache/trafficserver/tree/master/iocore/net/P_SSLNextProtocolAccept.h)
- [SSLNextProtocolAccept.cc](http://github.com/apache/trafficserver/tree/master/iocore/net/SSLNextProtocolAccept.cc)