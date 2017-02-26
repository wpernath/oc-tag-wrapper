# oc-tag-wrapper
Script that wraps oc tag to help with release management

This script has different functions.

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
