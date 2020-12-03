Tearing Down and Recreating Your Lab

If you get your Lab into a bad state, you might find the quickest way to get you back on track is to tear down and recreate your lab.

Tear Down

Start by deleting all instances:
```
multipass delete --all
```
Instances after they’re deleted are still stored in multipass, but in a deleted state. This is an implementation detail of multipass, this means the instances are gone from the virtualization layer, but still stored in multipass. To fully delete the instances (similar to a trash / recycle bin on desktop platforms), we must purge the instances:
```
multipass purge
```
