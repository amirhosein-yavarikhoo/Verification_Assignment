# Needed to convert make list into comma serperated string for cocotb
empty :=
space := $(empty) $(empty)
comma := ,
# -------------------------------------------------------------------------------
# NOT COCOTB ARGS
# -------------------------------------------------------------------------------

ROOT_DIR = $(shell pwd)

# Defining path
RTL_VERSION ?= rtl
RTL_DIR = $(ROOT_DIR)/../$(RTL_VERSION)

TB_DIR = $(ROOT_DIR)

# Define the test directory.
# The standard make target (see Simulation Targets below) are configured to execute any tests
# referenced by filename. So if you have $(TEST_DIR)/cl_marb_my_test.py you can
# run the test by
#
#    make cl_marb_my_test
#
# Additionaly a regression target is provided where
#
#    make regression
#
# will run all tests located in $(TEST_DIR)

TEST_DIR = tests
TEST_PATHS = $(wildcard $(TEST_DIR)/cl_marb_*_test.py)
TEST_LIST = $(patsubst $(TEST_DIR)/%.py,$(TEST_DIR).%,$(TEST_PATHS))

# because Makefile is in a different directory
export PYTHONPATH := $(TB_DIR)

LOCAL_PYTHON_BIN := $(shell which python)
PYTHON_PREFIX := $(shell $(LOCAL_PYTHON_BIN) -c "import sys; print(sys.base_prefix)")
PYTHON_VERSION := $(shell $(LOCAL_PYTHON_BIN) -c "import sys; print(f'python{sys.version_info.major}.{sys.version_info.minor}')")

export PYTHON_H_HOME = $(PYTHON_PREFIX)/include/$(PYTHON_VERSION)
export PYTHON_LIB = $(VIRTUAL_ENV)/lib/$(PYTHON_VERSION)

export PYUVM_LOG_LEVEL := INFO

# ------------------------------------------------------------------------------
# COCOTB ARGS
# ------------------------------------------------------------------------------

# Defaults
SIM ?= icarus
TOPLEVEL_LANG ?= verilog
# Verilog source files to include
VERILOG_INCLUDE_DIRS := $(RTL_DIR)
VERILOG_SOURCES := $(RTL_DIR)/pkg_mem_arb_types.sv
VERILOG_SOURCES += $(RTL_DIR)/apb_config_if.sv
VERILOG_SOURCES += $(RTL_DIR)/priority_sel.sv
VERILOG_SOURCES += $(RTL_DIR)/single_sort.sv
VERILOG_SOURCES += $(RTL_DIR)/mem_arb.sv
VERILOG_SOURCES += $(RTL_DIR)/mem_arb_wrapper.sv
VERILOG_SOURCES += $(RTL_DIR)/crc8.sv

# COCOTB_TOPLEVEL is the DUT wrapper
COCOTB_TOPLEVEL := mem_arb_wrapper

# RTL Parameters
ADDR_WIDTH ?= 8
DATA_WIDTH ?= 8

# Export vars for make subcalls
export RTL_VERSION SIM TOPLEVEL_LANG ADDR_WIDTH DATA_WIDTH

COMPILE_ARGS += -P$(COCOTB_TOPLEVEL).ADDR_WIDTH=$(ADDR_WIDTH)
COMPILE_ARGS += -P$(COCOTB_TOPLEVEL).DATA_WIDTH=$(DATA_WIDTH)


# Move the results generated inside the sim_build directory
COCOTB_RESULTS_FILE = $(shell pwd)/$(SIM_BUILD)/results.xml

# include cocotb's make rules to take care of the simulator setup
include $(shell cocotb-config --makefiles)/Makefile.sim

# ------------------------------------------------------------------------------

# To run our targets and cocotb targets
clean::
	@$(MAKE) local-clean -C $(TB_DIR)

ifeq ($(PYUVM_LOG_LEVEL), DEBUG)
    $(info PYUVM_LOG_LEVEL is ${PYUVM_LOG_LEVEL})
endif

# ------------------------------------------------------------------------------
# Simulation targets
# ------------------------------------------------------------------------------

# run test by name, e.g. make cl_marb_basic_test
%: tests/%.py
	$(MAKE) COCOTB_TEST_MODULES=$(TEST_DIR).$@ WAVES=1

.PHONY: regression $(COCOTB_RESULTS_FILE)
regression: COCOTB_TEST_MODULES = $(subst $(space),$(comma),$(TEST_LIST))
regression: WAVES = 1


# -------------------------------------------------------------------------------
# Because of PyVSC and coverage
# -------------------------------------------------------------------------------

# PyVSC (help debug)
export VSC_SOLVEFAIL_DEBUG = 0
export VSC_CAPTURE_SRCINFO = 0
export VSC_DEBUG = 0

THE_SIM_BUILD = $(TB_DIR)/$(SIM_BUILD)
COV_FILES = $(shell find $(THE_SIM_BUILD) -type f \( -iname "*.xml" \))

# https://pyucis.readthedocs.io/en/latest/commands.html
.PHONY: coverage-merge
coverage-merge:
	@echo ""
	@echo "======================================================"
	@echo "Merging coverage files..."
	@echo $(THE_SIM_BUILD)
	@echo $(COV_FILES)
	@echo "======================================================"
	@echo ""
	@$(VIRTUAL_ENV)/bin/pyucis merge --out $(THE_SIM_BUILD)/merge_cov.xml $(THE_SIM_BUILD)/*.xml

.PHONY: coverage-view
coverage-view: coverage-merge
	@echo ""
	@echo "======================================================"
	@echo "Opening PyUCIS-viewer"
	@echo "======================================================"
	@echo ""
	@$(VIRTUAL_ENV)/bin/pyucis-viewer $(THE_SIM_BUILD)/merge_cov.xml

.PHONY: coverage
coverage: coverage-view

# -------------------------------------------------------------------------------
# Clean-up target
# -------------------------------------------------------------------------------

.PHONY: local-clean
local-clean:
	@echo -e "\n------------------ Internal cleanup --------------------\n"
	# Remove all __pycache__ folders
	@rm -fvr $(shell find $(ROOT_DIR) -type d \( -iname '__pycache__' \);)

.PHONY: local-debug
local-debug::
	@echo -e "\n--------------------------------------\n"
	@echo DEBUG_MODE = $(DEBUG_MODE)

	@echo TEST_LIST = $(TEST_LIST)

	@echo PYTHONPATH = $(PYTHONPATH)
	@echo PYTHON_H_HOME = $(PYTHON_H_HOME)
	@echo "run 'where python' "
	@echo -e "\n--------------------------------------\n"
