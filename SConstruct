"""
This is a generic SCons script for running Sphinx (http://sphinx.pocoo.org).

Type 'scons -h' for help.  This prints the available build targets on your
system, and the configuration options you can set.

If you set the 'cache' option, the option settings are cached into a file
called '.sconsrc-sphinx' in the current directory.  When running
subsequently, this file is reread.  A file with this name is also read from
your home directory, if it exists, so you can put global settings there.

The script looks into your 'conf.py' file for information about the
project.  This is used in various places (e.g., to print the introductory
message, and create package files).

Here's some examples.  To build HTML docs:

   scons html

To create a package containing HTML and PDF docs, remembering the 'install'
setting:

   scons install=html,pdf cache=True package

To clean up everything:

   scons -c all
"""

# Script info.
__author__  = "Glenn Hutchings"
__email__   = "zondo42@googlemail.com"
__url__     = "http://bitbucket.org/zondo/sphinx-scons"
__license__ = "BSD"
__version__ = "0.4"

import sys, os

# Build targets.
targets = (
    ("html",      "make standalone HTML files"),
    ("dirhtml",   "make HTML files named index.html in directories"),
    ("pickle",    "make pickle files"),
    ("json",      "make JSON files"),
    ("htmlhelp",  "make HTML files and a HTML help project"),
    ("qthelp",    "make HTML files and a qthelp project"),
    ("devhelp",   "make HTML files and a GNOME DevHelp project"),
    ("epub",      "make HTML files and an EPUB file for bookreaders"),
    ("latex",     "make LaTeX sources"),
    ("texinfo",   "make Texinfo sources"),
    ("text",      "make text file for each RST file"),
    ("pdf",       "make PDF file from LaTeX sources"),
    ("ps",        "make PostScript file from LaTeX sources"),
    ("dvi",       "make DVI file from LaTeX sources"),
    ("changes",   "make an overview over all changed/added/deprecated items"),
    ("linkcheck", "check all external links for integrity"),
    ("doctest",   "run all doctests embedded in the documentation if enabled"),
    ("source",    "run a command to generate the reStructuredText source"),
)

# LaTeX builders.
latex_builders = {"pdf": "PDF", "ps": "PostScript", "dvi": "DVI"}

# List of target names.
targetnames = [name for name, desc in targets]

# Configuration cache filename.
cachefile = ".sconsrc-sphinx"

# User cache file.
homedir = os.path.expanduser('~')
usercache = os.path.join(homedir, cachefile)

# Configuration options.
config = Variables([usercache, cachefile], ARGUMENTS)

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
                 PathVariable.PathAccept),
    EnumVariable("pkgtype", "package type to build with 'scons package'",
                 "zip", ["zip", "targz", "tarbz2"], ignorecase = False),
    BoolVariable("cache", "whether to cache settings in %s" % cachefile, False),
    BoolVariable("debug", "debugging flag", False),
    ("genrst", "Command to regenerate reStructuredText source", None),
)

# Create a new environment, inheriting PATH to find builder program.  Also
# force LaTeX instead of TeX, since the .tex file won't exist at the right
# time to check which one to use.
env = Environment(ENV = {"PATH" : os.environ["PATH"]},
                  TEX = "latex", PDFTEX = "pdflatex",
                  tools = ['default', 'packaging'],
                  variables = config)
if 'PYTHONPATH' in os.environ:
    env['ENV']['PYTHONPATH'] = os.environ['PYTHONPATH']

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
genrst = env.get("genrst", None)

instdir = env["instdir"]
install = env["install"]
pkgtype = env["pkgtype"]

# Dump internals if debugging.
if debug:
    print "Environment:"
    print env.Dump()

# Get parameters from Sphinx config file.
sphinxparams = {}
execfile(sphinxconf, sphinxparams)

project = sphinxparams["project"]
release = sphinxparams["release"]
copyright = sphinxparams["copyright"]

try:
    texfilename = sphinxparams["latex_documents"][0][1]
except KeyError:
    texfilename = None

name2tag = lambda name: name.replace(" ", "-").strip("()")
project_tag = name2tag(project)
release_tag = name2tag(release)
package_tag = project_tag.lower() + "-" + release_tag.lower()

# Build project description string.
description = "%(project)s, release %(release)s, " \
               "copyright %(copyright)s" % locals()

Help(description + "\n\n")
help_format = "   %-10s  %s\n"

# Print banner if required.
if not any(map(GetOption, ("silent", "clean", "help"))):
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
%(builder)s -b %(name)s -d %(doctrees)s %(options)s %(srcdir)s %(targetdir)s
""".strip()

# Set up LaTeX input builder if required.
if texfilename:
    latexdir = Dir("latex", builddir)
    texinput = File(texfilename, latexdir)
    env.SideEffect(texinput, "latex")
    env.NoClean(texinput)

# Add build targets.
Help("Build targets:\n\n")

if genrst != None:
    source = env.Command('source', [], genrst, chdir = True)
    env.AlwaysBuild(source)
    env.Depends(srcdir, source)
else:
    source = env.Command(
        'source', [],
        '@echo "No reStructuredText generator (genrst) given."')

for name, desc in targets:
    target = Dir(name, builddir)
    targetdir = str(target)

    if name == 'source':
        pass
    elif name not in latex_builders:
        # Standard Sphinx target.
        targets = env.Command(name, sphinxconf,
                              sphinxcmd % locals(), chdir = True)
        env.Depends(targets, source)
        env.AlwaysBuild(name)
        env.Alias(target, name)
    elif texinput:
        # Target built from LaTeX sources.
        try:
            buildfunc = getattr(env, latex_builders[name])
        except AttributeError:
            continue

        filename = project_tag + "." + name
        outfile = File(filename, latexdir)

        targets = buildfunc(outfile, texinput)
        env.Depends(targets, source)

        # Copy built file to separate directory.
        target = File(filename, target)
        env.Command(target, outfile, Move(target, outfile), chdir = True)

        env.Alias(name, target)
    else:
        continue

    env.Clean(name, [target])
    env.Clean('all', target)

    if name == default: desc += " (default)"
    Help(help_format % (name, desc))

Clean('all', doctrees)
Default(default)

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
env.Command('uninstall', None, Delete(projectdir), chdir = True)
Help(help_format % ("uninstall", "uninstall documentation"))

# Add package builder.
packageroot = "-".join([project_tag, release_tag])
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

# Save local configuration if required.
if cache:
    config.Update(env)
    config.Save(cachefile, env)
