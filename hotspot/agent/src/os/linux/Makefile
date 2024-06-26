#
# Copyright (c) 2002, 2018, Oracle and/or its affiliates. All rights reserved.
# DO NOT ALTER OR REMOVE COPYRIGHT NOTICES OR THIS FILE HEADER.
#
# This code is free software; you can redistribute it and/or modify it
# under the terms of the GNU General Public License version 2 only, as
# published by the Free Software Foundation.
#
# This code is distributed in the hope that it will be useful, but WITHOUT
# ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or
# FITNESS FOR A PARTICULAR PURPOSE.  See the GNU General Public License
# version 2 for more details (a copy is included in the LICENSE file that
# accompanied this code).
#
# You should have received a copy of the GNU General Public License version
# 2 along with this work; if not, write to the Free Software Foundation,
# Inc., 51 Franklin St, Fifth Floor, Boston, MA 02110-1301 USA.
#
# Please contact Oracle, 500 Oracle Parkway, Redwood Shores, CA 94065 USA
# or visit www.oracle.com if you need additional information or have any
# questions.
#  
#

#
# This file has been modified by Loongson Technology in 2020. These
# modifications are Copyright (c) 2015, 2020, Loongson Technology, and are made
# available on the same license terms set forth above.
#

ARCH := $(shell if ([ `uname -m` = "ia64" ])  ; then echo ia64 ; elif ([ `uname -m` = "mips64el" ]) ; then echo mips64 ; elif ([ `uname -m` = "x86_64" ]) ; then echo amd64; elif ([ `uname -m` = "sparc64" ]) ; then echo sparc; else echo i386 ; fi )
GCC      = gcc

JAVAH    = ${JAVA_HOME}/bin/javah

SOURCES  = salibelf.c   \
        symtab.c        \
	libproc_impl.c  \
	ps_proc.c       \
	ps_core.c       \
	LinuxDebuggerLocal.c 

INCLUDES = -I${JAVA_HOME}/include -I${JAVA_HOME}/include/linux 

OBJS     = $(SOURCES:%.c=$(ARCH)/%.o) $(ARCH)/sadis.o

LIBS     = -lthread_db

CFLAGS   = -c -fPIC -g -D_GNU_SOURCE -D$(ARCH) $(INCLUDES) -I$(ARCH)

LIBSA = $(ARCH)/libsaproc.so

all: $(LIBSA)

$(ARCH):
       mkdir $(ARCH)

$(ARCH)/LinuxDebuggerLocal.o: LinuxDebuggerLocal.c
        $(JAVAH) -jni -classpath ../../../build/classes -d $(ARCH) \
		sun.jvm.hotspot.debugger.x86.X86ThreadContext \
		sun.jvm.hotspot.debugger.sparc.SPARCThreadContext \
		sun.jvm.hotspot.debugger.mips64.MIPS64ThreadContext \
		sun.jvm.hotspot.debugger.loongarch64.LOONGARCH64ThreadContext \
		sun.jvm.hotspot.debugger.amd64.AMD64ThreadContext \
		sun.jvm.hotspot.debugger.aarch64.AARCH64ThreadContext 
        $(GCC) $(CFLAGS) $< -o $@

$(ARCH)/sadis.o:  ../../share/native/sadis.c
        $(JAVAH) -jni -classpath ../../../build/classes -d $(ARCH) \
                sun.jvm.hotspot.asm.Disassembler
        $(GCC) $(CFLAGS) $< -o $@

$(ARCH)/%.o: %.c
        $(GCC) $(CFLAGS) $< -o $@

ifndef LDNOMAP
  LFLAGS_LIBSA = -Xlinker --version-script=mapfile
endif

# If this is a --hash-style=gnu system, use --hash-style=both
#   The gnu .hash section won't work on some Linux systems like SuSE 10.
_HAS_HASH_STYLE_GNU:=$(shell $(CC) -dumpspecs | grep -- '--hash-style=gnu')
ifneq ($(_HAS_HASH_STYLE_GNU),)
  LDFLAGS_HASH_STYLE = -Wl,--hash-style=both
endif
LFLAGS_LIBSA += $(LDFLAGS_HASH_STYLE)

LDFLAGS_NO_EXEC_STACK="-Wl,-z,noexecstack"
LFLAGS_LIBSA += $(LDFLAGS_NO_EXEC_STACK)

$(LIBSA): $(ARCH) $(OBJS) mapfile
        $(GCC) -shared $(LFLAGS_LIBSA) -o $(LIBSA) $(OBJS) $(LIBS)

test.o: test.c
	$(GCC) -c -o test.o -g -D_GNU_SOURCE -D$(ARCH) $(INCLUDES) test.c

test: test.o
	$(GCC) -o test test.o -L$(ARCH) -lsaproc $(LIBS)

clean:
        rm -fr $(ARCH)
