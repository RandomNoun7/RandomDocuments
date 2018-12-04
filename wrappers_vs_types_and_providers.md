# Tradeoffs of dsc_lite wrapper vs Types and Providers

## Summary

I have come to the conclusion that a module made of defined type wrappers around DSC_Lite resources is not the right solution for a Puppet supported module. This team has run into pain simply by taking one of our own supported modules as a dependency. Creating another module that depends not only on our module, but on third party modules and an intermediate configuration platform for its functionality is asking for trouble.

Given that an API now exists to leverage in the form of PowerShell cmdlets, I think the best, most supportable route to getting this done is via a new module that leverages real types and providers, and relies on Bolt tasks and plans to do the orchestration work.

## DSC_Lite Wrapper via Defined Types

### Pros

1. Relatively quick

    - Most of the real work of the configuration would be done by the DSC code written by third parties.

2. Bug fixes implemented by the DSC Module authors would easily be downloaded by customers and as long as the interfaces used by the module don't change, our module would automatically benefit from the fix.

3. Would result in classes that could automatically be recognized by PE and would be assignable to roles and profiles, without the end users having to wrap them in their own classes.

### Cons

1. Taking a dependency on a puppet module, even a Windows team supported module, in this case the DSC_Lite module, has caused this team pain in the past that would be nice to avoid.

2. Dependencies on community maintained DSC modules is a significant risk.

    - The current attempt at a DSC wrapper Clustering module has a dependency on our DSC module, which looks like it's currently shipping a bug. Their module is broken for now as a result, until we rev our DSC module and take up the fix.

    - Not only are we at risk for their bugs, but we are at the mercy of their release cadence for fixes. Even if the fix is known to be a pain point for our users, and we develop a fix and PR against the DSC module project, there is no telling if the PR will be accepted, or when a release might occur.

3. DSC on Windows, compared to native Puppet is a lossy intermediate layer.

    - While DSC_Lite works functionally as a configuration layer, it has problems with the way it reports results, and the fact that its failure modes aren't always great for Puppet.

    - For instance, the way the forge module for Clustering fails at the moment, is that it locks the Local Configuration Manager for that run, and possibly the next run as well. This means that not only does that module fail, but most likely any DSC code in that entire run after the failure will also most likely fail.

## New Types and Providers

### Pros

1. The integration will be much tighter, with no intermediate configuration layers and technologies.

    - Puppet will know exactly what operation has taken place and will report it faithfully.

    - Implementing control flow and logic will be much easier because we will be dealing in the native ruby code.

2. We will be dealing directly with the required PowerShell cmdlets. This is the lowest level API we have available for use. The alternative of using DSC resource is effectively just a higher level, less flexible API written by people who are not the platform maintainers.

3. Writing the code on our own means we can move quickly in response to customer needs and feedback because we are in control of the entire code base.

4. We will be in control of when to make possibly breaking changes for our customers. It is not uncommon for the community maintained DSC modules to rather casually ship breaking changes. If we write our own types and providers we are in control of when that should or should not happen.

### Cons

1. A lot of work.

    - I think I'm coming to the conclusion that this should be entirely it's own module. If you use the SQL module to install your SQL Instances, that's great, but fundamentally, installing and configuring a SQL Server, and configuring a cluster may not really be the same task. If we split this out we can make a module that is much easier to understand and maintain on it's own, rather than shoving it into an already fairly complicated module like SQLServer.

    - Either way, whether in its own module, or as a part of SQLServer, this will be a very significant engineering effort.

2. Did I mention this would take a while and would be a lot of work?