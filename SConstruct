### Generic SCons script for building Sphinx documentation.

# Author: Glenn Hutchings (zondo42@googlemail.com).
# License: BSD
# Version: 0.1

import os

# Sphinx targets.
targets = (
    ("html",      "make standalone HTML files"),
    ("dirhtml",   "make HTML files named index.html in directories"),
    ("pickle",    "make pickle files"),
    ("json",      "make JSON files"),
    ("htmlhelp",  "make HTML files and a HTML help project"),
    ("qthelp",    "make HTML files and a qthelp project"),
    ("latex",     "make LaTeX files"),
    ("changes",   "make an overview over all changed/added/deprecated items"),
    ("linkcheck", "check all external links for integrity"),
    ("doctest",   "run all doctests embedded in the documentation if enabled"),
)

# Options.
options = (
    ("config",    "conf.py", "sphinx configuration file"),
    ("srcdir",    ".",       "source directory"),
    ("builddir",  "_build",  "build directory"),
    ("doctrees",  None,      "doctrees directory (default: in build dir)"),
    ("paper",     None,      "LaTeX paper size (a4 or letter)"),
)

# Default target.
default = "html"

# Initialize help text.
help_usage = "Usage: scons [options] [target]\n"

# Set option values from arguments.
optvalues = {}

help_options = []
help_options.append("\n\nOptions (set by name=value):\n")

for name, value, desc in options:
    optvalues[name] = ARGUMENTS.get(name, value)
    if value: desc += " (default: %s)" % value
    help_options.append("   %-10s  %s" % (name, desc))

# Put doctrees in build dir unless otherwise specified.
if not optvalues["doctrees"]:
    builddir = optvalues["builddir"]
    optvalues["doctrees"] = os.path.join(builddir, "doctrees")

# Build sphinx command-line options.
sphinxopts = []

if optvalues["paper"]:
    sphinxopts.append("-D latex_paper_size=%s" % optvalues["paper"])

optvalues["sphinxopts"] = " ".join(sphinxopts)

# Build Sphinx command template.
sphinxcmd = """
sphinx-build -b %%s -d %(doctrees)s %(sphinxopts)s %(srcdir)s $TARGET
""".strip() % optvalues

# Create a new environment, inheriting all env variables (probably only
# need PATH, to find sphinx-build).
env = Environment(ENV = os.environ)

# Add standard sphinx targets.
help_targets = []
help_targets.append("\nAvailable targets:\n")

for name, desc in targets:
    target = Dir(name, builddir)

    env.Command(target, optvalues["config"], sphinxcmd % name)
    env.AlwaysBuild(target)
    env.Alias(name, target)
    env.Clean(name, [target])

    if name == default: desc += " (default)"
    help_targets.append("   %-10s  %s" % (name, desc))

# Set the default target.
Default(default)

# Set final help text.
Help(help_usage + "\n".join(help_targets) + "\n".join(help_options) + "\n")
