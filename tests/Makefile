# Simple makefile to run blockrom FTL unittests
# 
# maintained by thilo fromm <fromm@dresearch-fe.de>
#
#
# REQUIREMENTS
#
# 'test_harness.h', a C unit test mocking helper, is required to build
#  the tests. You can get it from https://github.com/t-lo/test_harness.
# 


TESTS = test-blockrom-init-exit			\
		test-blockrom-add				\
		test-blockrom-add-threaded		\
		test-blockrom-add-kzalloc-fail	\
		test-blockrom-readwritesect		\
		test-blockrom-remove

TEST_SRC = ${TESTS:%=%.c}

CC			  ?= gcc
BUILD_CC	  ?= ${CC}
BUILD_CFLAGS  ?= -Wall -Werror -I.
BUILD_LDFLAGS ?=

.PHONY: all clean run-tests

all: blockrom.c run-tests

run-tests: ${TESTS}
		@for t in ${^}; do 										\
			test -x $$t && ./$$t >$$t.log 2>&1 && echo "OK  : $$t" || 			\
				(echo "FAIL: $$t"; cat $$t.log; false; break)	\
		done
		@gcov -fb ${TEST_SRC}  1>coverage.log 2>&1 
		@grep "File 'blockrom.c'" coverage.log -A 4

clean:
		rm -f ${TESTS} *.gcda *.gcno *.log *.gcov blockrom.c

test-%: test-%.c
		${BUILD_CC} ${BUILD_CFLAGS} ${BUILD_LDFLAGS} -Iinclude --coverage -ftest-coverage -o ${@} ${<}

blockrom.c:
		cp ../src/drivers/mtd/blockrom.c .
