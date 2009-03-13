"""
This is a generic SCons script for running Sphinx (http://sphinx.pocoo.org).
"""

# Script info.
__author__  = "Glenn Hutchings"
__email__   = "zondo42@googlemail.com"
__url__     = "http://bitbucket.org/zondo/sphinx-scons"
__license__ = "BSD"
__version__ = "0.2"

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

# Configuration cache file.
cachefile = "conf-scons.py"

# Configuration variables.
config = Variables(cachefile, ARGUMENTS)

config.AddVariables(
    EnumVariable("default", "default build target", "html",
                 [name for name, desc in targets]),

    PathVariable("config", "sphinx configuration file", "conf.py"),

    PathVariable("srcdir", "source directory", ".",
                 PathVariable.PathIsDir),

    PathVariable("builddir", "build directory", "build",
                 PathVariable.PathIsDirCreate),

    PathVariable("doctrees", "place to put doctrees", None,
                 PathVariable.PathAccept),

    EnumVariable("paper", "LaTeX paper size", None,
                 ["a4", "letter"], ignorecase = False),

    ("tags", "comma-separated list of 'only' tags", None),
    ("builder", "program to run to build things", "sphinx-build"),
    ("options", "extra Sphinx options to use", None),

    BoolVariable("cache", "whether to cache variables", False),
)

# Create a new environment, inheriting PATH to find sphinx-build.
env = Environment(ENV = {"PATH" : os.environ["PATH"]},
                  variables = config)

# Get configuration values from environment.
sphinxconf = env["config"]
default = env["default"]

srcdir = env["srcdir"]
builddir = env["builddir"]
doctrees = env.get("doctrees", os.path.join(builddir, "doctrees"))

builder = env["builder"]
options = env.get("options", None)

paper = env.get("paper", None)
tags = env.get("tags", None)

# Get parameters from Sphinx config file.
sphinxparams = {}
execfile(sphinxconf, sphinxparams)

# Build project description string.
description = "%(project)s, release %(release)s, " \
               "copyright %(copyright)s" % sphinxparams

Help(description + "\n\n")

# Maybe print introduction.
if not GetOption("silent") and not GetOption("help"):
    print
    print "This is", description
    print

# Build sphinx command-line options.
opts = []

if tags:
    opts.extend(["-t %s" % tag for tag in tags.split(",")])

if paper:
    opts.append("-D latex_paper_size=%s" % paper)

if options:
    opts.append(options)

options = " ".join(opts)

# Build Sphinx command template.
sphinxcmd = """
%(builder)s -b %%s -d %(doctrees)s %(options)s %(srcdir)s $TARGET
""".strip() % locals()

# Add standard sphinx targets.
Help("Available targets:\n\n")

for name, desc in targets:
    target = Dir(name, builddir)

    env.Command(target, sphinxconf, sphinxcmd % name)
    env.AlwaysBuild(target)
    env.Alias(name, target)
    env.Clean(name, [target])

    if name == default: desc += " (default)"
    Help("   %-10s  %s\n" % (name, desc))

# Set the default target.
Default(default)

# Add config settings to help.
Help("\nConfiguration variables:")
for line in config.GenerateHelpText(env).split("\n"):
    Help("\n   " + line)

# Save build configuration.
if cachefile:
    config.Update(env)
    config.Save(cachefile, env)
