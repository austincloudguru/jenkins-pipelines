# jenkins-pipelines

First pass at Jenkins pipelines for spinnig up and shutting down terraform environments.  In order to work, it requires a jenkins-init script be placed in your terraform directory to facilitate using a remote config.

    #!/bin/bash -e
    
    # You must set PROJECT and BUCKET as environmental variables in your jenkins job.
    
    init() {
      if [ -d .terraform ]; then
        if [ -e .terraform/terraform.tfstate ]; then
          echo "Remote state already exist!"
          if [ -z $IGNORE_INIT ]; then
            exit 1
          fi
        fi
      fi
    
    
      terraform remote config \
        -backend=s3 \
        -backend-config="bucket=${BUCKET}" \
        -backend-config="key=${PROJECT}-${COMPONENT}-terraform.tfstate" \
        -backend-config="region=us-east-1"
    
    }
    
    while getopts "i" opt; do
      case "$opt" in
        i)
          IGNORE_INIT="true"
          ;;
      esac
    done
    
    shift $((OPTIND-1))
    
    init

