# Jupyter PySONIC
This repository packages PySONIC and MorphoSONIC in a JupyterLab environment on o²S²PARC.

JupyterLab with [PySONIC](https://github.com/tjjlemaire/PySONIC) and [MorphoSONIC](https://github.com/tjjlemaire/MorphoSONIC) for simulation of ultrasound neurostimulation, authored by Théo Lemaire.

## Kernels

A Python kernel, with PySONIC, MorphoSONIC and their dependencies (e.g. the NEURON simulator) is installed


## Information for developers

Building the docker image:

```shell
make build
```


Test the built image locally:

```shell
make run-local
```
Note that the `validation` directory will be mounted inside the service.


Raising the version can be achieved via one for three methods. The `major`,`minor` or `patch` can be bumped, for example:

```shell
make version-patch
```

If you already have a local copy of **o<sup>2</sup>S<sup>2</sup>PARC** running and wish to push data to the local registry:

```shell
make publish-local
```
## Citation

If this code base contributes to a project that leads to a scientific publication, please acknowledge this fact by citing:
- Lemaire, T., Neufeld, E., Kuster, N., and Micera, S. (2019). Understanding ultrasound neuromodulation using a computationally efficient and interpretable model of intramembrane cavitation. J. Neural Eng. [DOI](https://doi.org/10.1088/1741-2552/ab1685)
- Lemaire, T., Vicari E., Neufeld, E., Kuster, N., and Micera, S. (2021). MorphoSONIC: a morphologically structured intramembrane cavitation model reveals fiber-specific neuromodulation by ultrasound. iScience [DOI](https://doi.org/10.1016/j.isci.2021.103085)

## Testing

In a Jupyter notebook or terminal, run some of the code from the [MorphoSONIC repository](https://github.com/tjjlemaire/PySONIC)

## Changelog

### [2.0.0] - 2023-12-22
- New dynamics-sidecar version, from original version in GitLab
- Added MorphoSONIC