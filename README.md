# oc-tag-wrapper
Script that wraps oc tag to help with release management


## oc-tag options

    $ oc-tag

    Usage:
     oc-tag [-c] [-i <source-project>] [-s <image-stream-name>] -t <tag-name> [-o <target-project>] [-q <source-tag>]

    Tags the ImageStream of the <source-project> with <tag-name> in the <target-project>
      if -c is given, oc-tag checks what tags are set on latest ImageStream
      if -i is omitted, the current working project will be used
      if -o is omitted, the current working project will be used
      if -s is omitted, the first image stream will be used
      if -q is given, the <source-tag> tagged version is used to tag

      -i, --source-project    specify the source project to use or the current if omitted
      -s, --image-stream-name specify the name of the image stream or the first one if omitted
      -t, --tag-name          specify the name of the tag to use
      -o, --target-project    specify the name of the target project or use the current if omitted
      -q, --source-tag        specify the name of the source tag
      -c, --check-tag         print all Tags which point on 'latest'
      -h, --help			  print help and exit


## Make it easy to tag images
First of all it was created to simplify oc tag behavior as it is really to tag the right image. For this purpose, simply call

    $ oc-tag --image-stream-name simple-openshift-sinatra-sti --tag-name RELEASE_1_0

This tags the image stream named 'simple-openshift-sinatra-sti' in the current project to *RELEASE_1_0*. It also labels the image stream with the following label: *APP_VERSION=RELEASE_1_0*

## Help with release management

Assume you have a *development* project where you are doing your daily development tasks. Once you're ready to integrate and test, you want to tag the release and you want to have a project *testing* which points to that tagged image. 

    $ oc new-project testing
    $ oc-tag --source-project development --source-tag RELEASE_1_0 --target-project testing --tag-name latest --image-stream-name simple-openshift-sinatra-sti
    $ oc new-app simple-openshift-sinatra-sti:latest
    
You're first creating the *testing* project. Then you're tagging from RELEASE_1_0 release in *development* project to latest in *testing* project. Now you have a 'copy' of the RELEASE_1_0 tagged image stream available as latest in *testing*. The last line (which you only have to call once if you're starting with your tests in this stage) creates the app based on the image stream tagged latest.

Internally - as source-project and target-project differs and source-tag and tag-name are also different - it will also tag latest in target-project as RELEASE_1_0 and it will label it accordingly.

Now you can continue develop your code in *development* and once you're ready, you would release the next version.

    $ oc-tag --image-stream-name simple-openshift-sinatra-sti --tag-name RELEASE_1_1
    $ oc-tag --source-project development --source-tag RELEASE_1_1 --target-project testing --tag-name latest --image-stream-name simple-openshift-sinatra-sti
    
Now latest points to RELEASE_1_1, so OpenShift would start a new deployment in testing project and the label *APP_VERSION* is moved to RELEASE_1_1. 

Now within the OpenShift UI you can always see which version of your application is currently deployed. 


## Print all TAGs on latest ImageStream

Assume, you're having your App running since a long time. You now would like to know which version of the App is currently running. Of course, you've always properly tagged your running application with a meaningful name (RELEASE_1_0 etc.). 

oc describe is provides you with everything you want to know:

    $ oc describe is
    Name:			helloworld
    Namespace:		development
    Created:		2 hours ago
    Labels:			APP_VERSION=PRERELEASE_1_1
                app=helloworld
    Annotations:		openshift.io/generated-by=OpenShiftWebConsole
    Docker Pull Spec:	172.30.129.85:5000/development/helloworld
    Unique Images:		2
    Tags:			4

    latest
      pushed image

      * 172.30.129.85:5000/development/helloworld@sha256:d3122fda31b581b1fd09ee2a4fac0acc2c373635028de430b187bc9250f5fa28
          About an hour ago
        172.30.129.85:5000/development/helloworld@sha256:3dd31966935111ce624daf10409122d5ddd8127ac83a805d50d61d9b64c21a85
          2 hours ago

    PRERELEASE_1_1
      tagged from helloworld@sha256:d3122fda31b581b1fd09ee2a4fac0acc2c373635028de430b187bc9250f5fa28

      * 172.30.129.85:5000/development/helloworld@sha256:d3122fda31b581b1fd09ee2a4fac0acc2c373635028de430b187bc9250f5fa28
          38 minutes ago

    RELEASE_1_0
      tagged from helloworld@sha256:3dd31966935111ce624daf10409122d5ddd8127ac83a805d50d61d9b64c21a85

      * 172.30.129.85:5000/development/helloworld@sha256:3dd31966935111ce624daf10409122d5ddd8127ac83a805d50d61d9b64c21a85
          2 hours ago

    RELEASE_1_1
      tagged from helloworld@sha256:d3122fda31b581b1fd09ee2a4fac0acc2c373635028de430b187bc9250f5fa28

      * 172.30.129.85:5000/development/helloworld@sha256:d3122fda31b581b1fd09ee2a4fac0acc2c373635028de430b187bc9250f5fa28
          About an hour ago

But it is quite hard, to really understand which TAG is currently being used as 'latest'. 

    $ oc-tag -c
    oc-tag -c

    Image Stream name not given. Using 'helloworld'...
    Checks where 'latest' IS of helloworld points to
    'latest' points on PRERELEASE_1_1
    'latest' points on RELEASE_1_1
    
oc-tag -c helps here. As you can see, it returns a list of all Tags on 'latest'. 

