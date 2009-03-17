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

# List of target names.
targetnames = [name for name, desc in targets]

# Configuration cache file.
cachefile = "conf-scons.py"

# Configuration variables.
config = Variables(cachefile, ARGUMENTS)

config.AddVariables(
    EnumVariable("default", "default build target", "html", targetnames),
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
    ("opts", "extra builder options to use", None),
    ListVariable("install", "targets to install", ["html"], targetnames),
    PathVariable("instdir", "installation directory", "/usr/local/doc",
                 PathVariable.PathIsDirCreate),
    EnumVariable("pkgtype", "package type to build with 'scons package'",
                 "zip", ["zip", "targz", "tarbz2"], ignorecase = False),
    BoolVariable("cache", "whether to cache variables", False),
    BoolVariable("debug", "debugging flag", False),
)

# Create a new environment, inheriting PATH to find builder program.
env = Environment(ENV = {"PATH" : os.environ["PATH"]},
                  tools = ['default', 'packaging'],
                  variables = config)

# Get configuration values from environment.
sphinxconf = env["config"]
builder = env["builder"]
default = env["default"]

srcdir = env["srcdir"]
builddir = env["builddir"]
doctrees = env.get("doctrees", os.path.join(builddir, "doctrees"))

cache = env["cache"]
debug = env["debug"]

options = env.get("opts", None)
paper = env.get("paper", None)
tags = env.get("tags", None)

instdir = env["instdir"]
install = env["install"]
pkgtype = env["pkgtype"]

if debug:
    print "Environment:"
    print env.Dump()

verbose = not GetOption("silent") and not GetOption("help")

# Get parameters from Sphinx config file.
sphinxparams = {}
execfile(sphinxconf, sphinxparams)

project = sphinxparams["project"]
release = sphinxparams["release"]
copyright = sphinxparams["copyright"]

project_tag = project.replace(" ", "-")
package_tag = project_tag + "-" + release

# Build project description string.
description = "%(project)s, release %(release)s, " \
               "copyright %(copyright)s" % locals()

Help(description + "\n\n")
help_format = "   %-10s  %s\n"

# Maybe print introduction.
if verbose:
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
%(builder)s -b %(name)s -d %(doctrees)s %(options)s %(srcdir)s $TARGET
""".strip()

# Add build targets.
Help("Build targets:\n\n")

for name, desc in targets:
    target = Dir(name, builddir)

    env.Command(target, sphinxconf, sphinxcmd % locals())
    env.AlwaysBuild(target)
    env.Alias(name, target)
    env.Clean(name, [target])
    env.Clean('all', target)

    if name == default: desc += " (default)"
    Help(help_format % (name, desc))

Clean('all', doctrees)
Default(default)

# Add special PDF target.
latexdir = Dir("latex", builddir)
pdfdir = Dir("pdf", builddir)
pdffile = File(project_tag + ".pdf", pdfdir)
texfile = File(project_tag + ".tex", latexdir)

env.PDF(pdffile, texfile)
env.Alias("pdf", pdffile)
env.Requires(texfile, latexdir)

Help(help_format % ("pdf", "make PDF file from LaTeX sources"))

# Add installation targets and collect package sources.
Help("\nOther targets:\n\n")

Help(help_format % ("install", "install documentation"))
projectdir = os.path.join(instdir, project_tag)
sources = []

for name in install:
    source = Dir(name, builddir)
    sources.append(source)

    inst = env.Install(projectdir, source)
    env.Alias('install', inst)

    for node in env.Glob(os.path.join(str(source), '*')):
        filename = str(node).replace(builddir + os.path.sep, "")
        dirname = os.path.dirname(filename)
        dest = os.path.join(projectdir, dirname)
        inst = env.Install(dest, node)
        env.Alias('install', inst)

# Add uninstall target.
env.Command('uninstall', None, Delete(projectdir))
Help(help_format % ("uninstall", "uninstall documentation"))

# Add package builder.
packageroot = "-".join([project_tag, release])
archive, package = env.Package(NAME = project_tag, VERSION = release,
                               PACKAGEROOT = packageroot,
                               PACKAGETYPE = pkgtype,
                               source = sources)

env.AlwaysBuild(archive)
env.AddPostAction(archive, Delete(packageroot))
Help(help_format % ("package", "build documentation package"))

env.Clean('all', archive)

# Add config settings to help.
Help("\nConfiguration variables:")
for line in config.GenerateHelpText(env).split("\n"):
    Help("\n   " + line)

# Save build configuration if required.
if cache:
    config.Update(env)
    config.Save(cachefile, env)
