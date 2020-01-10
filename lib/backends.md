## Configuration of backends
Local and remote jobs are placed through the definition of one or more
backends that are defined in a configuration file located at `~/.sjef/`*suffix*`/backends.xml` and/or `/usr/local/etc/sjef/`*suffix*`/backends.xml`, where *suffix* is the file extension of the project bundle, and indicates the software package that will run on the backend.
The following fields normally need to be specified in order to define the backend.

- `name` The name to be used as a handle for the backend
- `host` Hostname, possibly with user name, for ssh communication with the remote.

The following field is optional.
- `cache` Directory on the remote that will host the cache.
The project is placed within that directory using the absolute path name on the local host, in order to avoid conflicts between projects. Note that the cache path does not contain the host name of the local host, since that can sometimes change, but as a consequence one should be aware of the potential of clashes between projects generated on different machines.
 If not specified, a sensible default for `cache` is chosen.

If jobs are to be run informally on the remote, this is all that is needed, provided that sjef is found in the path on the remote machine.  If jobs are instead to be launched using a batch system, the following additional fields need to be defined.
- `run_command` The command used to submit the job. This might simply be the software package executable , possibly appended with some options, or a script file that submits an appropriate batch job to launch the package. At job submission time, the input file name is appended to `run_command`.
- `run_jobnumber` A [regular expression](http://www.cplusplus.com/reference/regex/ECMAScript/) that extracts the job number from the output of `run_command`.
- `kill_command` A command that will kill the job, when the job number is appended.
- `status_command` A command that will query the status the job, when the job number is appended.
- `status_waiting` A [regular expression](http://www.cplusplus.com/reference/regex/ECMAScript/) that matches the output of _status_command_ if the job is waiting to run.
- `status_running` A [regular expression](http://www.cplusplus.com/reference/regex/ECMAScript/) that matches the output of _status_command_ if the job is running.

Within the definition of `run_command`, a simple keyword substitution mechanism is available:

- `{prologue text%param}` is replaced by the value of parameter `param` if it is defined, prefixed by `prologue text`. Otherwise, the entire contents between `{}` is elided.
- `{prologue %param:default value}` works similarly, with substitution of `default value` instead of elision if `param` is not defined.

The key/value pairs are normally specified through the library call to run the job (`sjef::Project::run()`, `pysjef.Project.run()`, `sjef run`) but are then cached in the properties of the project using the backend, so that the defaults for subsequent runs for the same project are the values of the parameters used most recently. 

Example:

```xml
<backends>
  <!-- there is a default backend always defined internally by the library, and it is equivalent to
<backend name="local" run_command="molpro"/>
-->
  <!-- informal immediate launching of Molpro on a neighbouring workstation -->
    <backend name="linux" host="user@host" cache="/tmp/sjef-backend" run_command="molpro"/>
    <backend name="linux2" host="linux2" cache="/tmp/peter/sjef-backend" run_command="myMolpro/bin/molpro"/>
  <!-- an example of a Slurm system, with qmolpro a wrapper that constructs a Molpro job script,
       and submits with srun -->
    <backend name="slurmcluster"
             host="{%user}@slurmcluster.somewhere.edu"
             cache="/scratch/{%user}/sjef-project"
             run_command="/software/molpro/release/bin/qmolpro {-t %t} {-n %n} {-m %m} {-G %G} {-q %q:compute}"
             run_jobnumber="Submitted batch job *([0-9]+)"
             kill_command="scancel"
             status_command="squeue -j"
             status_running=" (CF|CG|R|ST|S) *[0-9]" status_waiting=" (PD|SE) *[0-9]"
    />
</backends>
```

[//]: # ( @page backends sjef-project backends)