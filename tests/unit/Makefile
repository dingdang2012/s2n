#
# Copyright 2014 Amazon.com, Inc. or its affiliates. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License").
# You may not use this file except in compliance with the License.
# A copy of the License is located at
#
#  http://aws.amazon.com/apache2.0
#
# or in the "license" file accompanying this file. This file is distributed
# on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either
# express or implied. See the License for the specific language governing
# permissions and limitations under the License.
#

SRCS=$(wildcard *.c)
OBJS=$(SRCS:.c=.o)
TESTS=$(SRCS:.c=)
VALGRIND_TESTS=$(SRCS:.c=_valgrind)

.PHONY : all
all: $(TESTS)

include ../../s2n.mk

CRUFT += $(wildcard *_test)

ifeq ($(shell uname),Darwin)
    LIBS = -lpthread -lm
else ifeq ($(shell uname),FreeBSD)
    LIBS = -lthr -lcrypto
else
    LIBS = -lpthread -lm -ldl -lrt -lcrypto
endif

# Suppress the unreachable code warning, because tests involve what should be
# unreachable code
CFLAGS += -Wno-unreachable-code -I../../libcrypto-root/include/ -I../../ -I../../api/
LDFLAGS += -L../../lib/ -L../../libcrypto-root/lib -L../testlib/ -ltests2n -ls2n ${LIBS}

$(TESTS)::
	@${CC} ${CFLAGS} -o $@ $@.c ${LDFLAGS} 2>&1
	@DYLD_LIBRARY_PATH="../../lib/:../testlib/:$$DYLD_LIBRARY_PATH" LD_LIBRARY_PATH="../../lib/:../testlib/:$$LD_LIBRARY_PATH" ./$@

.PHONY : valgrind
valgrind: $(TESTS)
	(for test in *_test ; do DYLD_LIBRARY_PATH="../../lib/:../testlib/:$$DYLD_LIBRARY_PATH" LD_LIBRARY_PATH="../../lib/:../testlib/:$$LD_LIBRARY_PATH" valgrind --leak-check=full --run-libc-freeres=no -q --error-exitcode=0 --gen-suppressions=all --log-fd=2 --num-callers=40 --leak-resolution=high --log-file=$$test-valgrind.log --undef-value-errors=no --trace-children=yes ./$$test; done )
