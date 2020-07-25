## 1.rmi

## 2.Remote
```java
public interface Remote {}
```
Remote接口用来标识需要被远程调用的对象。任何远程对象都需要直接或间接实现该接口，这样对象内方法才可以被远程调用。<br/>
实现类可以实现一个或多个远程对象接口或者继承远程对象类。RMI包提供了UnicastRemoteObject和Activatable类来加强和简化远程对象类的创建。<br/>

## 3.UnicastRemoteObject
```java
public class UnicastRemoteObject extends RemoteServer {

    //暴露对象的端口
    private int port = 0;

    private RMIClientSocketFactory csf = null;

    private RMIServerSocketFactory ssf = null;

    private static final long serialVersionUID = 4974527148936298033L;
    
    protected UnicastRemoteObject() throws RemoteException
    {
        this(0);
    }
    
    protected UnicastRemoteObject(int port) throws RemoteException
    {
        this.port = port;
        exportObject((Remote) this, port);
    }

    protected UnicastRemoteObject(int port,
                                  RMIClientSocketFactory csf,
                                  RMIServerSocketFactory ssf)
        throws RemoteException
    {
        this.port = port;
        this.csf = csf;
        this.ssf = ssf;
        exportObject((Remote) this, port, csf, ssf);
    }

    private void readObject(java.io.ObjectInputStream in)
        throws java.io.IOException, java.lang.ClassNotFoundException
    {
        in.defaultReadObject();
        reexport();
    }

    public Object clone() throws CloneNotSupportedException
    {
        try {
            UnicastRemoteObject cloned = (UnicastRemoteObject) super.clone();
            cloned.reexport();
            return cloned;
        } catch (RemoteException e) {
            throw new ServerCloneException("Clone failed", e);
        }
    }

    private void reexport() throws RemoteException
    {
        if (csf == null && ssf == null) {
            exportObject((Remote) this, port);
        } else {
            exportObject((Remote) this, port, csf, ssf);
        }
    }

    @Deprecated
    public static RemoteStub exportObject(Remote obj) throws RemoteException
    {

        return (RemoteStub) exportObject(obj, new UnicastServerRef(true));
    }


    public static Remote exportObject(Remote obj, int port)
        throws RemoteException
    {
        return exportObject(obj, new UnicastServerRef(port));
    }

    public static Remote exportObject(Remote obj, int port,
                                      RMIClientSocketFactory csf,
                                      RMIServerSocketFactory ssf)
        throws RemoteException
    {

        return exportObject(obj, new UnicastServerRef2(port, csf, ssf));
    }


    public static boolean unexportObject(Remote obj, boolean force)
        throws java.rmi.NoSuchObjectException
    {
        return sun.rmi.transport.ObjectTable.unexportObject(obj, force);
    }


    private static Remote exportObject(Remote obj, UnicastServerRef sref)
        throws RemoteException
    {
        if (obj instanceof UnicastRemoteObject) {
            ((UnicastRemoteObject) obj).ref = sref;
        }
        return sref.exportObject(obj, null, false);
    }
}
```
UnicastRemoteObject提供了六个方法来暴露远程对象：
+ 继承UnicastRemoteObject类，并调用UnicastRemoteObject()构造函数；
+ 继承UnicastRemoteObject类，并调用UnicastRemoteObject(int port)构造函数；
+ 继承UnicastRemoteObject类，并调用UnicastRemoteObject(int port, RMIClientSocketFactory csf, RMIServerSocketFactory ssf)构造函数；
+ 调用exportObject(Remote remote)方法；已被移除；
+ 调用exportObject(Remote remote, int port)方法；
+ 调用exportObject(Remote remote, int port, RMIClientSocketFactory csf, RMIServerSocketFactory ssf)方法。










