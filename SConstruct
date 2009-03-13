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

# File containing configuration variables.
cachefile = "SConstruct.cache"

# Configuration variables.
config = Variables(cachefile, ARGUMENTS)

config.AddVariables(
    PathVariable("config", "sphinx configuration file", "conf.py"),

    PathVariable("srcdir", "source directory", ".",
                 PathVariable.PathIsDir),

    PathVariable("builddir", "build directory", "_build",
                 PathVariable.PathIsDirCreate),

    PathVariable("doctrees",  "place to put doctrees", None,
                 PathVariable.PathAccept),

    EnumVariable("paper", "LaTeX paper size", None,
                 ["a4", "letter"], ignorecase = False),
)

# Default target.
default = "html"

# Create a new environment, inheriting PATH to find sphinx-build.
env = Environment(ENV = {"PATH" : os.environ["PATH"]},
                  variables = config)

# Get configuration values from environment.
sphinxconf = env["config"]
srcdir = env["srcdir"]
builddir = env["builddir"]
doctrees = env.get("doctrees", os.path.join(builddir, "doctrees"))
paper = env.get("paper", None)

# Get parameters from Sphinx config file.
sphinxparams = {}
execfile(sphinxconf, sphinxparams)

# Build project description string.
description = "%(project)s, release %(release)s, " \
               "copyright %(copyright)s" % sphinxparams

Help(description + "\n\n")

Help("SCons builder for Sphinx, version %s\n\n" % __version__)

# Maybe print it now.
if not GetOption("silent") and not GetOption("help"):
    print
    print "This is", description
    print

# Build sphinx command-line options.
opts = []

if paper:
    opts.append("-D latex_paper_size=%s" % paper)

sphinxopts = " ".join(opts)

# Build Sphinx command template.
sphinxcmd = """
sphinx-build -b %%s -d %(doctrees)s %(sphinxopts)s %(srcdir)s $TARGET
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
Help("\nConfiguration variables:\n")
for line in config.GenerateHelpText(env).split("\n"):
    Help("   " + line + "\n")

# Save build configuration.
config.Update(env)
config.Save(cachefile, env)
