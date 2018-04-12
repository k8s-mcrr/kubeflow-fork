<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Developer Guide For Ksonnet Packages](#developer-guide-for-ksonnet-packages)
  - [Creating New Ksonnet Components](#creating-new-ksonnet-components)
  - [Testing changes to ksonnet components](#testing-changes-to-ksonnet-components)
    - [New Packages](#new-packages)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Developer Guide For Ksonnet Packages

## Creating New Ksonnet Components

Here are some instructions for creating new Kubeflow packages.

1. Decide whether the component belongs in a new ksonnet [package](https://github.com/ksonnet/ksonnet/blob/master/docs/concepts.md#package)

	- Packages correspond to subdirectories of [kubeflow/kubeflow](https://github.com/kubeflow/kubeflow/tree/master/kubeflow)
	- Here's a simple decision tree

		- If all Kubeflow deployments should include this package put it in core

			- Optional packages probably don't belong in core

		- Otherwise, if there's an existing package that it belongs with put
			it in that package 
		- If no existing package is a good fit create a new one

1. If you are creating a new package follow these steps

	- Copy new-package-stub to a new directory giving it an appropriate name
	- modify parts.yaml setting the following fields
		- name
		- description
	- modify registry.yaml adding an entry for the new package

1. Create a libsonnet file see [all.libsonnet](https://github.com/kubeflow/kubeflow/tree/master/kubeflow/new-package-stub/all.libsonnet) to define the prototypes and parts for your component

	- If you have an existing YAML manifest you can just convert that to json
	  and use that as a starting point for your parts

	- If you have a helm chart you can use the template command to output a YAML file

	  ```
	  helm template path/to/chart
	  ```

	- You can use [convert_manifest_to_jsonnet.py](https://github.com/kubeflow/kubeflow/blob/master/hack/convert_manifest_to_jsonnet.py) to easily convert a file containing YAML manifests to jsonnet

	- Typically you will want to make the following changes

		- Set the namespace for most components
		- Use params for any variables that should be easily overwritable
		- For helm packages substitute `params.name` for the name of the release

1. Create one or more prototype files in the prototypes directory for your package

	- Use [tf-prototype.jsonnet](https://github.com/kubeflow/kubeflow/tree/master/kubeflow/new-package-stub/prototypes/tf-prototype.jsonnet) as a template.
	
## Testing changes to ksonnet components

The easiest way to test ksonnet components is to follow the normal instructions for setting up a 
ksonnet app to deploy kubeflow.

Then replace the directory `vendor/kubeflow` with a symbolic link to `${GIT_KUBEFLOW}/kubeflow`

```
ln -sf ${GIT_KUBEFLOW}/kubeflow  ${APP_DIR}/vendor/kubeflow
```

this way changes to your .libsonnet files will automaticaly be reflected in your components.

If you make changes to prototypes you need to regenerate the prototype. You can just delete the `.jsonnet`
file in your app's component directory and then regenerate the component using `ks generate`. 
If you use the same name you will preserve the values of any parameters you have set. 
ksonnet will print a warning but it works; e.g.

```
rm -rf ${APP_DIR}/components/kubeflow-core.jsonnet 
ks generate kubeflow-core kubeflow-core
INFO  Writing component at 'components/kubeflow-core'
ERROR Component parameters for 'kubeflow-core' already exists
```

### New Packages

If you are dealing with a new package you need to modify `app.yaml` in your ksonnet application
and add an entry in the libraries section for your new package.