# Working with DANDI

DANDI provides access to an archive that stores cellular neurophysiology
datasets. We refer to such datasets as `Dandisets`.

1. A `Dandiset` is organized in a structured manner to help users and
software tools interact with it.
1. Each `Dandiset` has a unique persistent identifier that you can use to go directly
to the `Dandiset` (e.g., [https://identifiers.org/DANDI:000004](https://identifiers.org/DANDI:000004)).
You can use this identifier to cite the `Dandiset` in your publications or provide
direct access to a `Dandiset`.

## DANDI components

### The DANDI Web application

The [DANDI Web application](https://dandiarchive.org/) allows you to:

1. Browse `Dandisets`.
1. Search across `Dandisets`.
1. Create an account to register a new `Dandiset` or gain access to
[the Dandihub analysis platform](#the-dandihub-analysis-platform).
1. Add collaborators to your `Dandiset`.
1. Retrieve an `API key` to perform data upload to your `Dandisets`.
1. Publish versions of your `Dandisets`.

If you want to learn more, go to [Working with DANDI Archive web](./11_using_dandi_web.md)

### The DANDI Python client

The [DANDI Python client](https://pypi.org/project/dandi/) allows you to:

1. Download `Danidsets` and individual subject folders or files.
1. Organize your data locally before upload.
1. Upload `Dandisets`.

### The Dandihub analysis platform

[Dandihub](https://hub.dandiarchive.org) provides a Jupyter environment to
interact with the DANDI archive. To use the hub, you will need to register an
account using [the DANDI Web application](#the-dandi-web-application). Please
note that `Dandihub` is not intended for significant computation, but provides a
place to introspect `Dandisets` and files.

## Downloading from DANDI

You can download entire `Dandisets` or single files.

### Downloading a file

#### Using the Web application

Each `Dandiset` has a `View Data` option. This provides a folder-like view to
navigate a `Dandiset`. Any file in the `Dandiset` has a download icon next to it.
You can click this icon to download a file to your device where you are browsing
or right click to get the download URL of the file. You can then use this URL
programmatically or in other applications such as the [NWB Explorer](https://nwbexplorer.opensourcebrain.org/)
or in a [Jupyter notebook on Dandihub](https://hub.dandiarchive.org).

#### Using the Python CLI

First install the Python client using `pip install dandi` in a Python 3.7+
environment.


1. Downloading a `Dandiset`, e.g.:
`dandi download DANDI:000023`
1. Downloading data for a specific subject from a dandiset
(names of the subjects could be found on the gui.dandiarchive.org website or by running `dandi ls -r DANDI:000023`), e.g.:
`dandi download https://api.dandiarchive.org/api/dandisets/000023/versions/draft/assets/?path=sub-P10HMH` or a specific version by replacing `draft` with it (e.g. `0.210914.1900` in the case of this dandiset)
1. Downloading a specific file from a dandiset (a link for the specific file could be found on the gui.dandiarchive.org website), e.g.:
`dandi download https://api.dandiarchive.org/api/dandisets/000023/versions/0.210914.1900/assets/1a93dc97-327d-4f9c-992d-c2149e7810ae/download/`

Hint: `dandi download` supports a number of resource identifiers to point to dandiset, folder, or file.  Providing an incorrect URL (e.g. `dandi download wrongurl`) will provide a list of supported identifiers.

## Create an account on DANDI

To create an account on DANDI, you will need to.

1. [Create a Github account](https://github.com/) if you don't have one.
1. Using your Github account [register a DANDI account](https://gui.dandiarchive.org/#/user/register).
1. You will receive an email acknowledging activation of your account within 24
hours. You can now login to DANDI using the Github by clicking on the login
button.

## Uploading a Dandiset

DANDI provides a production server (http://dandiarchive.org/) for real data and an identical staging server 
(https://gui-staging.dandiarchive.org/) for test data.
When you create a dandiset, a permanent ID is automatically assigned to it.
To prevent the production server from being inundated with test dandisets, we encourage developers to develop 
against the staging server. The below instructions will alert you to where the commands for interacting with these 
two different servers differ slightly. 

1. Setup
    - If you do not have a DANDI account, please [create an account](#create-an-account-on-dandi)
    - Log in to DANDI and copy your API key. Click on your user initials on the
    top right after logging in. Production (dandiarchive.org) and staging (gui-staging.dandiarchive.org) servers have different API keys and different logins.
    - Locally
        - Create a Python environment (not required, but strongly recommended, e.g., [miniconda](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-with-commands),
         [virtualenv](https://docs.python.org/3/library/venv.html)
        - Install the DANDI CLI into your Python environment

                pip install -U dandi

        - Store your API key somewhere that the CLI can find it; see ["Storing
          Access Credentials"](#storing-access-credentials) below.

1. Data upload/management workflow
    1. Register a dandiset to generate an identifier. You will be asked to enter
      basic metadata, a name (title) and description (abstract) for your dataset.
      Click `NEW DANDISET` in the Web application (top right corner) after logging in. 
      After you provide name and description, the dataset identifer will be created, 
      we will call this `<dataset_id>`.
    1. NWB format:
        1. Convert your data to NWB 2.1+ in a local folder. Let's call this `<source_folder>`.
        We suggest beginning the conversion process using only a small amount of data so that common issues may be spotted earlier in the process.
        This step can be complex depending on your data. Feel free to [reach out to
        us for help](https://github.com/dandi/helpdesk/discussions).
        1. Check your files for [NWB Best Practices](https://nwbinspector.readthedocs.io/en/dev/best_practices/best_practices_index.html) by installing
        the [NWBInspector](https://nwbinspector.readthedocs.io/en/dev/user_guide/user_guide.html) (`pip install -U nwbinspector`) and running

                nwbinspector <source_folder> --config dandi
                
        1. Thoroughly read through the NWBInspector report and try to address as many issues as possible. **DANDI will prevent validation and upload of any issues
        labeled as level 'CRITICAL' or above when using the `--config dandi` option.**
        We recommend regularly running the inspector early in the process to generate the best NWB files possible.
        Please note that some autodetected violations, such as `check_data_orientation`, may be safely ignored in the event that the data is confirmed to be in the correct form; this can be done using either the `--ignore <name_of_check_to_suppress>` flag or a config file. See [the NWBInspector CLI documentation](https://nwbinspector.readthedocs.io/en/dev/user_guide/command_line_usage.html) for more details and other options, or type `nwbinspector --help`.
        If the report is too large to efficiently navigate in your console, a report can be saved using

                nwbinspector <source_folder> --config dandi --report-save-path <report_location>.txt
        
        1. Once your files are confirmed to adhere to the Best Practices, perform an official validation of the NWB files by running: `dandi validate <source_folder>`. 
        **If you are having trouble with validation, make sure the conversions were run with the most recent version of `PyNWB` and `MatNWB`.**
        1. Preparing a dataset folder for upload:

                dandi download https://dandiarchive.org/dandiset/<dataset_id>/draft
                cd <dataset_id>
                dandi organize <source_folder> -f dry
                dandi organize <source_folder>
                dandi upload  # for staging: dandi upload -i dandi-staging

        1. Add metadata by visiting your dandiset landing page: 
       `https://dandiarchive.org/dandiset/<dataset_id>/draft` and clicking on the `METADATA` link.
        1. Use the dandiset URL in your preprint directly, or download it using the dandi CLI:
            `dandi download https://dandiarchive.org/dandiset/<dataset_id>/draft`
    1. BIDS format:
        1. Please [reach out to Dandi team for help](https://github.com/dandi/helpdesk/discussions).

## Storing Access Credentials

By default, the DANDI CLI looks for an API key in the `DANDI_API_KEY`
environment variable.  To set this on linux or osx, in run

```bash
export DANDI_API_KEY=personal-key-value
```
(note that there are no spaces around the "=").

If this is not set, the CLI will look up the API
key using the [keyring](https://github.com/jaraco/keyring) library, which
supports numerous backends, including the system keyring, an encrypted keyfile,
and a plaintext (unencrypted) keyfile.

- You can store your API key where the `keyring` library can find it by using
  the `keyring` program: Run `keyring set dandi-api-dandi key` and enter the
  API key when asked for the password for `key` in `dandi-api-dandi`.

- You can set the backend the `keyring` library uses either by setting the
  `PYTHON_KEYRING_BACKEND` environment variable or by filling in [the `keyring`
  library's configuration file](https://github.com/jaraco/keyring#configuring).
  IDs for the available backends can be listed by running `keyring --list`.  If
  no backend is specified in this way, the library will use the available
  backend with the highest priority.

If the API key isn't stored in either the `DANDI_API_KEY` environment variable
or in the keyring, the CLI will prompt you to enter the API key, and then it
will store it in the keyring.  This may cause you to be prompted further; you
may be asked to enter a password to encrypt/decrypt the keyring, or you may be
asked by your OS to confirm whether to give the DANDI CLI access to the
keyring.

- If the DANDI CLI encounters an error while attempting to fetch the API key
  from the default keyring backend, it will fall back to using an encrypted
  keyfile (the `keyrings.alt.file.EncryptedKeyring` backend).  If the keyfile
  does not already exist, the CLI will ask you for confirmation; if you answer
  "yes," the `keyring` configuration file (if does not already exist; see
  above) will be configured to use `EncryptedKeyring` as the default backend.
  If you answer "no," the CLI will exit with an error, and you must store the
  API key somewhere accessible to the CLI on your own.

## Publish a Dandiset

**🛠 Work in progress 🛠**

## Acquiring Debugging Information

In the event that something goes wrong while using the `dandi` client, the
first place to check for more information so that you can [file a quality bug
report](https://github.com/dandi/dandi-cli/issues) is the logs.  Every dandi
command records a copy of its logs in a logfile, the location of which is
reported to the user when the command finishes running.  The location of the
logs varies by platform:

- Linux: `~/.cache/dandi-cli/log` or `$XDG_CACHE_HOME/dandi-cli/log`
- macOS: `~/Library/Logs/dandi-cli`
- Windows XP: `C:\Documents and Settings\<username>\Local Settings\Application Data\dandi\dandi-cli\Logs`
- Windows Vista: `C:\Users\<username>\AppData\Local\dandi\dandi-cli\Logs`

Logs are named with a combination of the time at which the `dandi` command
started running and the process ID of the command.

Recent versions of `dandi` include all possible debugging information in the
logs, but if you're using an older version, only log messages that were printed
to the user when the command ran are recorded.  As a result, in order to get
complete debugging information, you may have to rerun the problematic command,
this time increasing the logging level by passing `-l DEBUG` or `--log-level
DEBUG` on the command line.  Note that this option goes between the main
`dandi` command and the name of the subcommand:

    # Right:
    dandi -l DEBUG upload

    # Wrong:
    dandi upload -l DEBUG

In addition, many commands can be put into a developer-specific mode for
showing raw progress information instead of fancy progress bars.  For the
`delete`, `organize`, `upload`, and `validate` commands, this can be done by
setting the `DANDI_DEVEL` environment variable and passing `--devel-debug` to
the command, like so:

    DANDI_DEVEL=1 dandi upload --devel-debug

For the `download` command, the equivalent is the `-f debug`/`--format debug`
option:

    dandi download -f debug

More advanced users who are familiar with [the Python
debugger](https://docs.python.org/3/library/pdb.html) can instruct `dandi` to
automatically open the debugger if any errors occur by supplying the `--pdb`
option to the command.  Like the `-l`/`--log-level` option, the `--pdb` option
must be placed between `dandi` and the name of the subcommand.
