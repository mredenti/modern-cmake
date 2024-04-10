---
title: Finding Dependencies 
teaching: 10
exercises: 10
---

:::::::::::::::::::::::::::::::::::::: questions 

- How to find and consume TPL?

::::::::::::::::::::::::::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::: objectives

- Ex... 
  
::::::::::::::::::::::::::::::::::::::::::::::::


As a project grows it will start to likely rely on third party dependencies. For instance, it might require specific libraries, such as FFTW3, to be already available on the file system which perhaps define a range of things including targets, functions, variables...

**Notes:** 

- [ ] A later episode will explore CMake's support for fetching, building and installing missing dependencies. 
- [ ] A later episode will explore CMake's support for preparing a project for being found by other projects. 

CMake provides features which enable projects to find things and to be found by or incorporated into other projects:

# CMake `find_package()`


# Find CMake configuration files



## CMake Modules 


### What to do when neither options above are available?

CMake modules add the ability to use `pkg-config` to provide information about external packages, while other modules facilitate writing package files for other projects to consume.


::::::::::::::::::::::::::::::::::::: keypoints 

- fd

::::::::::::::::::::::::::::::::::::::::::::::::