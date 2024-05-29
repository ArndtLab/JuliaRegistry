# The ArndtLab Julia Registry

## Register a new package or tag a release via ArndtLabRegistryTools

* if tagging a new version of an already registered package bump the version number in its `Project.toml`
* git commit and push the package
* startup `julia` in the project dir (w/o `--proj`), and run the following code. Double check wether the `tree_hash` points to the right commit you want to register.

```julia
using ArndtLabRegistryTools; arndtlabregister()
```



## Register a new package or tag a release 

* if tagging a new version of an already registered package bump the version number in its `Project.toml`
* git commit and push the package
* startup `julia` in the project dir (w/o `--proj`), and run the following code. Double check wether the `tree_hash` points to the right commit you want to register.

```julia
using RegistryTools, Pkg

function myregister(registry_repo, package_path = ".", tree_hash = nothing)

	package_repo = string(chomp(read(Cmd(`git remote get-url --all origin`, dir = package_path), String)))
	pkg = Pkg.Types.read_project(joinpath(package_path, "Project.toml"))
	if tree_hash == nothing
		possible_hashes = readlines(Cmd(`git log --pretty=format:'%T %s'`, dir = package_path))

		println(join(possible_hashes, "\n"))
		
		tree_hash = possible_hashes[1] |> split |> first
		@show tree_hash
	end

	r = RegistryTools.register(
		package_repo, pkg, tree_hash,
		registry = registry_repo,
		registry_deps = [RegistryTools.DEFAULT_REGISTRY_URL],
		push = true
	)
end


registry_repo = "git@github.com:ArndtLab/JuliaRegistry.git"

myregister(registry_repo)
```

* Now visit the website of the repo of your registry to create and merge a PR.

* Afterwards git-tag the new version with and clean up:
```bash
v=vN.M.K
```

```bash
git tag -a $v -m \"$v\"
git push origin $v

rm -rf registries/
```
* Go to repo on github and pubish a release clicking on the 3 dots to the right in the tags listing


## Create a Registry

First create an empty repo at github.com or an (on prem) alternative, e.g.`username/MyRegistry.git`. In the following `"git@github.com:username/MyRegistry.git"` has to be adjusted appropriately.   Then start `julia`:

```julia
using RegistryTools, Pkg, UUIDs

registry_name = "JuliaRegistry"
registry_repo = "git@github.com:ArndtLab/JuliaRegistry.git"

path = "."
uuid = string(UUIDs.uuid4())


rd = RegistryTools.RegistryData(registry_name, uuid, repo = registry_repo)
RegistryTools.write_registry(joinpath(path, "Registry.toml"), rd)
```

Now git init, add, commit and push the `Registry.toml` to the repo from your shell (adjusting the remote):

```bash
git init
git add Registry.toml
git commit -m "first commit"
git remote add origin git@github.com:ArndtLab/JuliaRegistry.git
git push -u origin master
```

The local git repo and `Registry.toml` can be trashed. The Registry is still empty - to be useful you have to make the package manger aware of it and register packages in it.


## Add a registry to the package manager

To make the package manager aware of your new (private) Registry add it:

```
pkg> registry add git@github.com:ArndtLab/JuliaRegistry.git
```
If the repo is public one might also use:
```
pkg> registry add https://github.com/ArndtLab/JuliaRegistry.git
```
