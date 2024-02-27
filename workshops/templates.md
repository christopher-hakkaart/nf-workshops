# Streamline your Nextflow pipeline development with nf-core templates

As the ecosystem evolves, so does the template.
Different versions of nf-core tools have different templates. 

Check the versions of nextflow is 23.10.1:

```
nextflow -v
```

Check the version of nf-core is 2.13:

```
nf-core --version
```

> This is written for the most recent nf-core tools 2.13

## Getting started

nf-core tooling is for both users and developers and has commands that complement each other. 

```
nf-core
```

The template is generated using the `nf-core create` command:

```
nf-core create
```

The template is flexible and can be customized to include different features.

Prompts to create pipeline:

```
? Workflow name mypipeline
? Description My pipeline
? Author Chris
? Do you want to customize which parts of the template are used? (y/N) y
? Pipeline prefix myorg
Skip template areas?
   ○ GitHub hosting
   ○ GitHub CI
   ● GitHub badges
   ○ iGenomes config
   ● nf-core/configs
```

Template areas

- GitHub hosting
  - Files required for GitHub hosting of the pipeline
    - `.github/`
    - `.gitignore`
- GitHub CI:
  - Files required for GitHub continuous integration tests
    - `.github/workflows/` 
- GitHub badges
  - GitHub badges in the `README.md` file.
- iGenomes config
  - Pipeline options related to iGenomes.
    - `conf/igenomes.config`
- nf-core/configs
  - Repository options that integrate nf-core config profiles.

> GitHub badges and nf-core/configs were skipped in the example above.

A pipeline template named `myorg-mypipeline` was created.

Code for submitting this to GitHub was suggested and should be modified to submit the code to the repository.

```
cd /Users/chris/workspace/myorg-mypipeline
git remote add origin git@github.com:<USERNAME>/<REPO>.git
git push --all origin
```

> `<USERNAME>` is the GitHub handle and `<REPO>` is the repository name. The repository name is public and my GitHub credentials were already prepared.

Three branches were pushed to GitHub.

```
remote: Resolving deltas: 100% (10/10), done.
To https://github.com/<USERNAME>/<REPO>.git
 * [new branch]      TEMPLATE -> TEMPLATE
 * [new branch]      dev -> dev
 * [new branch]      main -> main
```

The `TEMPLATE` branch is especially important as it is used by the `nf-core sync` command and will help integrate template changes in the pipeline.

## Anatomy of the nf-core template

The template can be overwhelming but a complete understanding isn't required to get started.

Important parts:

1. The name and version of the pipeline are in the right place throughout this template. This is important for pipeline development as the name to identify the pipeline and will be incorporated into some of the logging and reporting features.
   - e.g., `workflows/mypipeline.nf`

2. The template comes with the git and config files that work together to help manage your pipeline development with GitHub and CI testing.
   - e.g., `.github/workflows`, `.gitignore`, and `.prettier.yml`

3. The template comes with a set of files for linting tests with `nf-core lint` and is designed to help you write good code and help keep all of the moving parts in the pipeline in sync.
  - e.g., `nf-core.yml`

4. The template comes with a schema that is used for parameter validation and documentation to help you write good code and to help others run and understand the pipeline.
   - e.g., `nextflow_schema.json`

5. A consistent format for every nf-core pipeline giving a sense of familiarity and plugs into the wider nf-core ecosystem.
   - e.g., integration with nf-core/modules and schema is rendered in Seqera platform.

## New features as of nf-core/tools version 2.13

The groovy code that use to live in the `lib` folder has been moved to `subworkflows/`.

Now easier to find, modify, and test code.

Also more modular and is paving the way for a more flexible template in the future.

## A pipeline out of the box

The template is a working pipeline that can use the test profile (`conf/test.config`) for development purposes.

```
nextflow run myorg-mypipeline -profile test,docker --outdir results
```

There are additional profiles for software management (e.g., `docker`, `singularity`, & `conda`).

Test profile is important for template CI testing that runs with new commits are made to GitHub and ensures no breaking changes.

## Plugin components

Can pull in components from the nf-core ecosystem (e.g., modules and subworkflows).

```
cd myorg-mypipeline
nf-core modules install
```

Follow the prompt to pull in `fastp` module.

```
? Tool name: fastp
```

Modules is automatically injected into `modules/nf-core/`.

Gives the code to be added to `workflows/mypipeline.nf`.

```
include { FASTP } from '../modules/nf-core/fastp/main'
```

The module comes packaged with everything needed to run with a profile locally and on the cloud.

Can find inputs required to plug module into pipeline in `modules/nf-core/fastp/main.nf`.

```
input:
tuple val(meta), path(reads)
path  adapter_fasta
val   save_trimmed_fail
val   save_merged
```

Can check what these are on [nf-core website](https://nf-co.re/modules/fastp).

Code for `workflows/mypipeline.nf`

```
//
// MODULE: Run Fastp
//
FASTP (
    ch_samplesheet,
    [],
    false,
    false
)
```

Modules are being tracked in `modules.json`.

Can also use a private repository of modules using `--git-remote` in `nf-core modules install` command.

There are over 1000 modules and 50 subworkflows that are available from the nf-core repository.

## Linting is a passion

There are lots of moving parts and linting helps check your development practises.

```
nf-core lint
```

There are warnings to keep you honest and help with development (e.g., TODOs) and reproducibility (changes between local and remote).

```
╭───────────────────────╮
│ LINT RESULTS SUMMARY  │
├───────────────────────┤
│ [✔] 159 Tests Passed  │
│ [?]  20 Tests Ignored │
│ [!]  24 Test Warnings │
│ [✗]   0 Tests Failed  │
╰───────────────────────╯
```

Editing the `fastp` module will cause the linting to fail (e.g., by adding an `s` to the end of `json`).

```
tuple val(meta), path('*.json')           , emit: jsons
```

Running `nf-core lint` again will now throw an error.

```
╭─ [✗] 1 Module Test Failed ─────────────────────────────────────────────────────────────────────╮
│              ╷                               ╷                                                 │
│ Module name  │ File path                     │ Test message                                    │
│╶─────────────┼───────────────────────────────┼─────────────────────────────────────────────────│
│ fastp        │ modules/nf-core/fastp/main.nf │ Local copy of module does not match remote      │
│              ╵                               ╵                                                 │
╰────────────────────────────────────────────────────────────────────────────────────────────────╯
```

The difference between the local and remote can be patched with nf-core tooling.

```
nf-core modules patch
```

The prompt can be followed to patch the `fastp` module.

```
? Module name: fastp
```

A patch file is created in the fastp module directory

```
...
INFO     'modules/nf-core/fastp/tests/main.nf.test.snap' is unchanged
INFO     'modules/nf-core/fastp/tests/tags.yml' is unchanged
INFO     'modules/nf-core/fastp/tests/nextflow.config' is unchanged
INFO     'modules/nf-core/fastp/tests/main.nf.test' is unchanged 
INFO     Patch file of 'modules/nf-core/fastp' written to 'modules/nf-core/fastp/fastp.diff' 
```

All tests by the `nf-core lint` command will now pass.

## Pipeline schema

The schema should be updated with details for every parameter.

Adding a parameter (`skip_fastp`) to skip the `FASTP` process will break the linting if the schema is not updated. 

Updating `workflows/mypipeline.nf`.

```
//
// MODULE: Run Fastp
//
if(!params.skip_fastp) {
    ch_samplesheet = FASTP (
        ch_samplesheet,
        [],
        false,
        false
    )
}
```

Updating the `nextflow.config` file in the template repository.

```
    // Skips steps
    skip_fastp                = false
```

Without updating the schema the linting will fail.

```
nf-core lint
```

```
╭─ [✗] 1 Pipeline Test Failed ───────────────────────────────────────────────────────────────────╮
│                                                                                                │
│ schema_params: Param skip_fastp from nextflow config not found in nextflow_schema.json         │
│                                                                                                │
╰────────────────────────────────────────────────────────────────────────────────────────────────╯
```

```
nf-core schema build
```

## TEMPLATE syncs

The template evolves as the ecosystem evolves.

New templates are release semi-regularly and the nf-core tooling helps incorporate these changes through syncs with the TEMPLATE branch.

```
git add .
git commit -m "Added fastp"
git push
```

```
nf-core sync
```

The tooling merges updates suggesting a git command.

```
cd /Users/chris/workspace/myorg-mypipeline                            
git merge TEMPLATE 
```

## Bump the version across

The pipeline version can be bumped across the template using nf-core tooling.

```
nf-core bump-version 1.0
```

## Update GitHub

Changes should be pushed to GitHub.

```
git add .
git commit -m "Version release"
git push
```
