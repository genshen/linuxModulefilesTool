## Linux Environment Module Files Tool
this is a linux environment module files tool,it gennerate module files with configure file and template files.  
what is [Environment Module](http://modules.sourceforge.net/),it provides dynamic modification of a user's environment via **modulefiles**.

## Features
+ you can just write one template file for different versions of one software package.

## How does it word
the go program first read configure file:modulefiles.yaml which is located at current folder.  
note:the modulefiles.yaml is written via yaml configure language,you can find more detail at [http://www.yaml.org/](http://www.yaml.org/)

then the go program generate modulefiles one by one:  
travel configure.items in file modulefiles.yaml; value *module_name,tag,version,description,home and additional* will be send to template where value *template_name* refers to.  
note1: the template is located by {{.configure.template_prefix}}+{{.configure.items[i].template_name}}  
note2: if a value (assume it's **example_key:exmaple_value**) is send a template,then the text **{{.example_key}}** in template will replaced by **example_value**

## example
modulefiles.yaml:  
```yaml
# we use yaml rule to write configure file, more details at:http://yaml.org
last_modified: 2017-04-22
last_modified_by: genshen
configure:
    #release_prefix and template_prefix must end with '/'
    release_prefix: release/
    template_prefix: templates/
    items:
        cmake/3.8.0:  #usually it's same as module_name
            module_name: cmake/3.8.0
            module_alias: cmake/3.8.0
            tag: cmake  #used for conflict in modules
            version: 3.8.0
            description: '"Set cmake3.8.0 for use."'
            home: /opt/tools/cmake/3.8.0
            template_name: cmake_3.8.0.txt
            additional:
                system: linux_x86_64
    #some other items below
```

Add template file: templates/cmake_3.8.0.txt  
```
#%Module1.0#####################################################################
##{{.info}}
##
## modules {{.module_name}}
##
## modulefiles/{{.module_name}}. Written by genshen, April-23-2017
##
proc ModulesHelp { } {
        global version homedir
        puts stderr "\t{{.module_name}} - sets the Environment for cmake {{.version}}"
        puts stderr "\n\tThe homedir is {{.home}}"
        puts stderr "\n\n\t#####   DO NOT MOVE OR MODIFY THE PATH OF CMAKE #####"
}

module-whatis	 {{.description}}

conflict {{.tag}}

prepend-path    PATH            ${{.home}}/bin
```
note: you can change the value of {{.info}} in main.go via const **Info**  

then the template file will be rendered to modulefiles(located at release/cmake/3.8.0):
```
#%Module1.0#####################################################################
##[Info]DO NOT modify this file directly,this file was generated by module files tools, by genshen(genshenchu@gmail.com)
##
## modules cmake/3.8.0
##
## modulefiles/cmake/3.8.0. Written by genshen, April-23-2017
##
proc ModulesHelp { } {
        global version homedir
        puts stderr "\tcmake/3.8.0 - sets the Environment for cmake 3.8.0"
        puts stderr "\n\tThe homedir is /opt/tools/cmake/3.8.0"
        puts stderr "\n\n\t#####   DO NOT MOVE OR MODIFY THE PATH OF CMAKE #####"
}

module-whatis	 "Set cmake3.8.0 for use."

conflict cmake

prepend-path    PATH            $/opt/tools/cmake/3.8.0/bin
```
then copy files in release/* to /etc/modulefiles/  
now you can load or unload cmake/3.8.0 module via environment module,e.g. *module load cmake/3.8.0*   
all ok!

## how to build
ensure you have install [golang](https://golang.org) and configure GOROOT and GOPATH
#### dependency packages
```
go get gopkg.in/yaml.v2
```
just run:
```
go build main.go
```

## License
[MIT](https://opensource.org/licenses/MIT)