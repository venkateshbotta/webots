# Copyright 1996-2020 Cyberbotics Ltd.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

ifeq ($(WEBOTS_HOME),)
WEBOTS_HOME_PATH = ../../..
else
null :=
space := $(null) $(null)
WEBOTS_HOME_PATH=$(subst $(space),\ ,$(strip $(subst \,/,$(WEBOTS_HOME))))
endif

include $(WEBOTS_HOME_PATH)/resources/Makefile.os.include
PYTHON_COMMAND ?= python
PYTHON_VERSION := $(shell $(PYTHON_COMMAND) --version 2>&1 | sed 's/.* \([0-9]\).\([0-9]\).*/\1.\2/')
PYTHON_SHORT_VERSION = $(subst .,,$(PYTHON_VERSION))
PYTHON_FULL_VERSION := $(subst Python ,,$(shell $(PYTHON_COMMAND) --version 2>&1))
INTERFACE       = controller.i
SWIG            = swig
SWIG_OPTS       = -c++ -python -outdir $(WEBOTS_CONTROLLER_LIB_PATH)/python$(PYTHON_SHORT_VERSION)/
WEBOTS_INCLUDES = -I$(WEBOTS_HOME_PATH)/include/controller/cpp -I$(WEBOTS_HOME_PATH)/include/controller/c
WRAPPER         = $(INTERFACE:.i=$(PYTHON_SHORT_VERSION).cpp)
WRAPPER_OBJECT  = $(WRAPPER:.cpp=.o)
PYOUT           = $(addprefix $(WEBOTS_CONTROLLER_LIB_PATH)/python$(PYTHON_SHORT_VERSION)/,$(INTERFACE:.i=.py))
PYTHON_PATH_SETUP := $(shell mkdir -p $(WEBOTS_CONTROLLER_LIB_PATH)/python$(PYTHON_SHORT_VERSION))

ifeq ($(OSTYPE),windows)
PYTHON_HOME    := $(subst \ ,$(space),$(dir $(subst $(space),\ ,$(shell which python 2> /dev/null))))
C_FLAGS         = -c -O -Wall -Wno-maybe-uninitialized
LD_FLAGS        = -shared
LIBS            = -L"$(PYTHON_HOME)libs" -lpython$(PYTHON_SHORT_VERSION) -L$(WEBOTS_CONTROLLER_LIB_PATH) -lController -lCppController
LIBOUT          = $(addprefix $(WEBOTS_CONTROLLER_LIB_PATH)/python$(PYTHON_SHORT_VERSION)/_,$(INTERFACE:.i=.pyd))
ifneq (,$(findstring 2.,$(PYTHON_VERSION)))
C_FLAGS        += -DMS_WIN64
endif
PYTHON_INCLUDES = -I"$(PYTHON_HOME)include"
LIBCONTROLLER   = $(WEBOTS_CONTROLLER_LIB_PATH)/Controller.dll
LIBCPPCONTROLLER= $(WEBOTS_CONTROLLER_LIB_PATH)/CppController.dll
endif

ifeq ($(OSTYPE),darwin)
ifeq ($(PYTHON_SHORT_VERSION), 27)
PYTHON_PATH ?= /System/Library/Frameworks/Python.framework/Versions/$(PYTHON_VERSION)
PYTHON_PYMALLOC =
else
PYTHON_PATH ?= /Library/Frameworks/Python.framework/Versions/$(PYTHON_VERSION)
ifeq ($(PYTHON_SHORT_VERSION), 38)
PYTHON_PYMALLOC =
else
PYTHON_PYMALLOC = m
endif
endif
BREW_PYTHON_PATH = /usr/local/Cellar/python/$(PYTHON_FULL_VERSION)/Frameworks/Python.framework/Versions/$(PYTHON_VERSION)
PYTHON_BIN       = $(PYTHON_PATH)/bin/
C_FLAGS          = -c -Wall -fPIC -mmacosx-version-min=$(MACOSX_MIN_SDK_VERSION) -Wno-uninitialized
ifeq ($(findstring llvm-g++,$(shell ls -lF $(shell which c++ 2> /dev/null))),)
C_FLAGS        += -Wno-self-assign
endif
LD_FLAGS        = -dynamiclib -install_name @rpath/lib/controller/python$(PYTHON_SHORT_VERSION)/_$(INTERFACE:.i=.dylib) -Wl,-rpath,@loader_path/../../.. -compatibility_version 1.0 -current_version 1.0.0 -mmacosx-version-min=$(MACOSX_MIN_SDK_VERSION)
LIBS            = -L"$(PYTHON_PATH)/lib" -L$(WEBOTS_CONTROLLER_LIB_PATH) -lController -lCppController -lpython$(PYTHON_VERSION)
LIBOUT          = $(addprefix $(WEBOTS_CONTROLLER_LIB_PATH)/python$(PYTHON_SHORT_VERSION)/_,$(INTERFACE:.i=.so))
PYTHON_INCLUDES = -isystem "$(PYTHON_PATH)/include/python$(PYTHON_VERSION)$(PYTHON_PYMALLOC)"
LIBCONTROLLER   = $(WEBOTS_CONTROLLER_LIB_PATH)/libController.dylib
LIBCPPCONTROLLER= $(WEBOTS_CONTROLLER_LIB_PATH)/libCppController.dylib
ifneq (,$(wildcard $(BREW_PYTHON_PATH)))
PYTHON_INCLUDES += -I"$(BREW_PYTHON_PATH)/include/python$(PYTHON_VERSION)$(PYTHON_PYMALLOC)"
LIBS            += -L"$(BREW_PYTHON_PATH)/lib"
endif
endif

ifeq ($(OSTYPE),linux)
C_FLAGS         = -c -Wall -fPIC -Wno-unused-but-set-variable -Wno-maybe-uninitialized
LD_FLAGS        = -shared
LIBS            = -L$(WEBOTS_CONTROLLER_LIB_PATH) -lController -lCppController
LIBOUT          = $(addprefix $(WEBOTS_CONTROLLER_LIB_PATH)/python$(PYTHON_SHORT_VERSION)/_,$(INTERFACE:.i=.so))
PYTHON_INCLUDES = -I"/usr/include/python$(PYTHON_VERSION)"
LIBCONTROLLER   = $(WEBOTS_CONTROLLER_LIB_PATH)/libController.so
LIBCPPCONTROLLER= $(WEBOTS_CONTROLLER_LIB_PATH)/libCppController.so
endif

SWIG_EXISTS     = $(shell which $(SWIG) 2> /dev/null)

TARGET          = $(PYOUT) $(LIBOUT)

ifeq (, $(shell which $(PYTHON_COMMAND) 2> /dev/null))
release debug profile:
	@echo -e "# \033[0;33m$(PYTHON_COMMAND) not installed, skipping Python API\033[0m"
else ifeq ($(SWIG_EXISTS),)
release debug profile:
	@echo -e "# \033[0;33mSWIG not installed, skipping Python API\033[0m"
else
release debug profile: $(TARGET)

$(PYOUT) $(WRAPPER):$(INTERFACE) $(LIBCONTROLLER) $(LIBCPPCONTROLLER)
	$(SWIG) $(SWIG_OPTS) $(WEBOTS_INCLUDES) -o $(WRAPPER) $<

$(LIBOUT):$(WRAPPER_OBJECT) $(LIBCONTROLLER) $(LIBCPPCONTROLLER)
	$(CXX) $(LD_FLAGS) $< $(LIBS) -o "$@"

$(LIBCONTROLLER):
	@echo "$(LIBCONTROLLER) doesn't exist"

$(LIBCPPCONTROLLER):
	@echo "$(LIBCPPCONTROLLER) doesn't exist"

$(WRAPPER_OBJECT):$(WRAPPER)
	$(CXX) $(C_FLAGS) $(WEBOTS_INCLUDES) $(PYTHON_INCLUDES) $< -o $@
endif

clean:
	rm -rf *.o *.cpp $(WEBOTS_CONTROLLER_LIB_PATH)/python*
