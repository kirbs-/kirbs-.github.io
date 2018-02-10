---
layout: page
title: Install XGBoost on El Capitan via pip
order: 1
---
Installing [XGboost](https://github.com/dmlc/xgboost) is straight forward with pip. However, if installation fails with  `XGBoostLibraryNotFound: Cannot find XGBoost Libarary in the candicate path, did you install compilers and run build.sh in root path?` This error is caused pip build requiring gcc 5. To fix run:

1. `brew install gcc@5`
2. `env CC=gcc-5 CXX=g++-5 pip install xgboost`
