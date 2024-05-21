Github Branch : https://github.com/Cray-HPE/istio/commits/work-1.19.10
(cherry picked the commits and rebased everything with modifying the correct version strings and some more modifications)
 

Build failures 


1. https://jenkins.algol60.net/blue/organizations/jenkins/Cray-HPE%2Fistio/detail/work-1.19.10/1/pipeline

Error: stat /work/samples/extauthz/cmd/extauthz: directory not found

Possible solution :

1. To keep extauthz:
Dont drop the samples/extauthz

2. To remove extauthz:

STANDARD_BINARIES:=./istioctl/cmd/istioctl \
  ./pilot/cmd/pilot-discovery \
  ./pkg/test/echo/cmd/client \
  ./pkg/test/echo/cmd/server \
  ./samples/extauthz/cmd/extauthz \
  ./operator/cmd/operator \
  ./tools/bug-report

  remove ./samples/extauthz/cmd/extauthz \ from Makefile.core.mk

TODO: Ask whether to keep extauthz svc

# Went with 1st option

Build failure 1.1

In Make Docker step its pushing to localhost:5000/pilot:1.19.10-20240520202132_0039b0a done

Why localhost:5000 ?

And consequently it couldn't pull the build from dockerhub for retagging in the next retag step

Change docker registry to istio instead of localhost:5000
