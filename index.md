<img src="https://licensezero.com/logo.svg" class="logo">

# License Zero Developer's Guide

License Zero is a game plan for financially sustaining independent work on open software.  License Zero lets you make a new deal with users of software you maintain:

> Use my software freely for open source or noncommercial work.  Otherwise, buy a license online to support me.

It's the model of [GitHub](https://github.com/pricing), [Travis CI](https://travis-ci.com/plans), and other leading developer services, adapted for independent library, framework, and tools developers.  Finally.

This model also makes it easy to support developers creating open software you use and rely on at work.  License Zero is not a donation platform or a backer-rewards market.  If other developers' work helps you do your job better or more efficiently, supporting them through License Zero ought to be a reimbursable expense.

This is a guide to License Zero, and the project's primary documentation for developers.  If you're interested in using License Zero, either to support your work or to support others, read this guide.  If you find errors or ways to improve it, send a [pull request](https://github.com/licensezero/licensezero-guide).

This guide describes what License Zero is, and how it works.  It is _not_ a substitute for legal advice about whether a particular license fits your needs or work.  The License Zero [terms of service](https://licensezero.com/terms/service) apply to this guide, too.

## Contents

- [Read This First](#read-this-first)
- [Public Licenses](#public-licenses)
  - [Comparing Public Licenses](#comparing-public-licenses)
  - [License Politics](#license-politics)
- [Private Licenses](#private-licenses)
- [Waivers](#waivers)
- [Relicensing](#relicensing) 
  - [Permissive License](#permissive-license)
  - [Relicense Agreement](#relicense-agreement)
- [licensezero.com](#licensezero.com)
  - [Stripe Connect](#stripe-connect)
  - [Identifiers](#identifiers)
  - [Cryptography](#cryptography)
  - [Project Pages](#project-pages)
  - [Pricing Graphics](#pricing-graphics)
- [Artless Devices](#artless-devices)
  - [Service Provider](#service-provider)
  - [Licensing Agent](#licensing-agent)
  - [Toolmaker](#toolmaker)
- [Command Line Interface](#command-line-interface)
  - [Identify Yourself](#identify-yourself)
  - [As a Contributor](#as-a-contributor)
  - [As a User](#as-a-user)
- [Ecosystem Support](#ecosystems)
- [Contributions](#contributions)
  - [Parallel Licensing](#parallel-licensing)
  - [Stacked Licensing](#stacked-licensing)
- [Complementary Approaches](#complementary-approaches)

## <a id="read-this-first">Read This First</a>

```python
def licensing(user, project):
  # See "Public Licenses" below.
  if user.meets_conditions_of(project.public_license):
    project.public_license.permit(user)
  else:
    # See "Private Licenses" below.
    private_license = user.private_license_for(project)
    if (
      private_license is not None and
      user.meets_conditions_of(private_license)
    ):
      private_license.permit(user)
    # See "Waivers" below.
    else:
      waiver = user.waiver_for(project)
      if (
        waiver is not None and
        not waiver.expired
      ):
        waiver.permit(user)
      # See "licensezero.com" below.
      else:
        license_zero.buy_private_license(user, project)
```

As an independent software developer, you control who can use your software, and under what terms.  License Zero `LICENSE` terms give everyone broad permission to use and build with your software, as long as they either limit commercial use or share work they build on yours back as open source, depending on the terms you choose.  Users who can't meet those conditions need private licenses.

[licensezero.com](https://licensezero.com) can sell private licenses on your behalf.  A [command line interface](https://www.npmjs.com/package/licensezero) makes it easy for users to buy all the private licenses they need for an [npm](https://www.npmjs.com)-based project at once, with a single credit card transaction.  [Stripe](https://stripe.com) processes payments directly to an account in your name.  The same command line interface makes it easy for you to sign up, start offering private licenses for sale, and set pricing.

This "dual licensing" model is not new.  [MySQL](https://www.mysql.com/about/legal/licensing/oem/) pioneered it decades ago, and important projects like [Qt](https://www1.qt.io/licensing/) and [MongoDB](https://www.mongodb.com/community/licensing) continue it, successfully, today.  License Zero evolves the dual licensing model by making it useful for more kinds of software, and by making it practical for independent developers who can't or don't want to set up companies.

## <a id="public-licenses">Public Licenses</a>

License Zero starts where you exercise your power as the owner of intellectual property in your work: in your project's `LICENSE` file.  You might currently use [The MIT License](https://spdx.org/licenses/MIT), [a BSD license](https://spdx.org/licenses/BSD-2-Clause), or a similar open source license there now.  License Zero offers you two alternatives:

1.  [The License Zero Noncommercial Public License (L0&#x2011;NC)](https://licensezero.com/licenses/noncommercial) works a bit like a [Creative Commons NonCommercial license](https://creativecommons.org/licenses/by-nc/4.0/), but for software.  L0&#x2011;NC gives everyone broad permission to use your software, but limits commercial use to a short trial period of 32 days.  When a commercial user's trial runs out, they need to buy a private license or stop using your software.

2.  [The License Zero Reciprocal Public License (L0&#x2011;R)](https://licensezero.com/licenses/reciprocal) works a bit like [AGPL](https://www.gnu.org/licenses/agpl-3.0.html), but requires users to release more of their own code, in more situations.  L0&#x2011;R requires users who change, build on, or use your work to create software to release that software as open source, too.  If users can't or won't release their work, they need to buy a private license that allows use without sharing back.

Both L0 public licenses are short and readable.  You should [read](https://licensezero.com/licenses/noncommercial) [them](https://licensezero.com/licenses/reciprocal).

### <a id="comparing-public-licenses">Comparing Public Licenses</a>

The two License Zero public licenses aren't just worded differently.  They achieve different results.  Abstracting them a bit:

```python
def noncommercial_license:
  if commercial_user:
    if within_trial_period:
      return 'free to use'
    else:
      return 'need to buy a private license'
  else:
    return 'free to use'

def reciprocal_license:
  if building_software:
    if release_software_as_open_source:
      return 'free to use'
    else:
      return 'need to buy a private license'
  else:
    return 'free to use'
```

Consider a few user scenarios, and how they play out under different License Zero public licenses:

1.  A developer employed at a for-profit company wants to use a License Zero library in their company's proprietary web app.

    - If the library is licensed under L0&#x2011;NC, the developer can only use the library for 32 days under its public license.  Then they need to buy a private license.

    - If the library is licensed under L0&#x2011;R, the developer needs to buy a private license to use it in their web app at all.  They can't use the library under its public license, because they won't meet its conditions by releasing their web app as open source.

2.  A developer employed at a for-profit company wants to use a License Zero library in the data synchronization software their company ships with voice recorders.  They plan to release the sync software as open source.

    - If the library is licensed under L0&#x2011;NC, the developer can only use the library for 32 days under the public license.  Then they need to buy a private license.  The private license allows the developer to "sublicense", or pass down, their permission to use the library for commercial purposes to their company and its customers.

    - If the library is licensed under L0&#x2011;R, the developer can use the library in their sync software under the public license, as long as they actually release the sync software as open source.  The company's customers can use the sync software, with the library, for any purpose.

3.  A developer employed at a for-profit company wants to use a License Zero video player application to show commercials in their office lobby.

    - If the application is licensed under L0&#x2011;NC, the developer can only use the application for 32 days under the public license.  Then they need to buy a private license.

    - If the application is licensed under L0&#x2011;R, the developer is free to use the application for as long as they like under the public license.

L0&#x2011;NC allows users to build and use closed and proprietary software with your work, as long they use it for noncommercial purposes.  L0&#x2011;R allows users to build and use only open source software with your work, even for very profit-driven purposes.  Many noncommercial software users are happy to make their work open source, but many make closed software, too.  Many for-profit companies make closed software, but many also make and release open source software, too.

License Zero was inspired by imbalances in the relationship between open source developers and users.  Both License Zero public licenses represent new deals between developers and users, designed to redress that imbalance.  It isn't clear yet which public license approach is best, either overall or for any particular kind of software.  Before you pick a public license, make a few notes about how you think users will use and build on your software.  Then think about how they will play out under L0&#x2011;NC, and then again under L0&#x2011;R.

A key consideration for applications is whether you want to require users to buy private licenses for commercial use.  Say you write an image manipulation program.   L0&#x2011;R would allow anyone to use your program for any purpose, including commercial purposes, so long as they don't modify your program or build on it to make closed source.  That's unlikely for an image manipulation program, as opposed to a software library or a software development tool.  On the other hand, L0&#x2011;NC would require users to buy a private license after 32 days.

### <a id="license-politics">License Politics</a>

The differences between the public licenses, especially on commercial use, reflect some political differences that you should be aware of.

L0&#x2011;NC is not an "open source" or "free software" license as many community members define those terms, because it discriminates against commercial use.  Source for L0&#x2011;NC can still be published and developed online, using many popular services like [GitHub](https://github.com) and [npm](https://www.npmjs.com).  But many in those communities will not accept it as part of their movements, and perhaps criticize you for referring to it as "open source" or "free software".

L0&#x2011;R, on the other hand, was written to conform to the [Open Source Definition](https://opensource.org/osd), and proposed to the [Open Source Initiative's license-review mailing list](http://lists.opensource.org/pipermail/license-review_lists.opensource.org/) for approval in September of 2017.  Approval has been controversial, in part because L0&#x2011;R goes further than existing licenses in when and what code it requires be released as open source.  It's our position that L0&#x2011;R conforms to the Open Source Definition as written, resembles licenses that OSI has approved in the past, and that software under L0&#x2011;R is therefore open source software.

L0&#x2011;R is probably not a "free software" license [as defined by the Free Software Foundation](https://www.gnu.org/philosophy/free-sw.html).  FSF's definition of free software requires granting freedoms to run, copy, distribute, study, change and improve software.  But it also recognizes that some conditions on those freedoms can enhance software freedom overall by ensuring that others receive source code and freedom to work with it.  However, FSF's definition of free software admits only conditions on the freedom to share modified versions with others, not other freedoms.  That partially explains why the Open Source Initiative [approved RPL&#x2011;1.5](https://opensource.org/licenses/RPL-1.5), a thematic predecessor of L0&#x2011;R, while FSF [considers RPL non-free](https://www.gnu.org/licenses/license-list.en.html#RPL).

In the end, your software is yours to license.  These politics may or may not matter to you, and they may or may not matter to your users or potential users.

## <a id="private-licenses">Private Licenses</a>

Users who can't meet the conditions of the License Zero public license for a project can buy a private license that allows them to use the project without meeting those conditions.  License Zero publishes a [form private license](https://licensezero.com/licenses/private), and sells private licenses to uses on developers' behalf.

Each private license grants the buyer broad permission under copyright and patent law to use the software, in language a bit more like what a company legal department would expect.  Otherwise, the legal effect is much like that of a permissive public license, except the private license applies only to the person who bought it, from the date they bought it.

Private licenses give the buyer limited ability to pass their licenses on, or sublicense, others.  Generally speaking, buyers can pass their licensees on to others if they build software with significant additional functionality with the License Zero code.  They can pass on only the right to use the License Zero as part of the new program they've created, and to maintain it.  They _can't_ pass on their license build different programs with the software.  In other words, buyers can sublicense the "user" parts of their private licenses, but not the "developer" part of their private licenses.  Other developers who want to develop new programs, commercially or as closed source, need to buy their own private licenses.

Private licenses last forever, and cover all contributions that the developer makes with the same [project identifier](#identifiers) listed in the private license.  Developers don't promise to do any new work, or make it available under the same project identifier, by selling private licenses.

## <a id="waivers">Waivers</a>

Waivers work a bit like freebie private licenses.  Developers can use the [command line interface](#command-line-interface) to generate signed [waivers](https://licensezero.com/licenses/waiver) for specific people that let them out of public license conditions limiting commercial use or requiri open source release.  Developers can generate waivers that last only a set number of days, or that last forever.  The [command line interface](#command-line-interface) treats waivers just like private licenses for purposes of quoting licenses a users needs for a project.

You might like to issue waivers to reward contributors to your project who make their work available under a permissive license, for [parallel licensing](#parallel-licensing), extend the free trial period for projects under L&#x2011;NC terms, or to resolve a question about whether a particular use will trigger the commercial-use time limit. 

## <a id="relicensing">Relicensing</a>

License Zero allows developers to set a price for changing the public license terms for their contributions to a project to those of [The License Zero Permissive Public License (L0&#x2011;P)](https://licensezero.com/licenses/permissive).  This is called "relicensing".

### <a id="permissive-license">Permissive License</a>

[L0&#x2011;P](https://licensezero.com/licenses/permissive) is a highly permissive open source software license, much like [The MIT License](https://spdx.org/licenses/MIT) and [the two-clause BSD license](https://spdx.org/licenses/BSD-2-Clause), but easier to read and more legally complete.  It gives everyone who receives a copy of your software permission under copyright and patent law to work with and built on it in any way they like, as long as they preserve your license information in copies they give to others and refrain from suing users of your project for violating patents on it.

Relicensing contributions to a project under L0&#x2011;P removes any reason for users to purchase private licenses for those contributions.  Under the [agency terms](https://licensezero.com/terms/agency) that developers must agree to in order to offer private licenses through License Zero, they must retract a project for sale through the API if they relicense it.

### <a id="relicense-agreement">Relicense Agreement</a>

The [relicense agreement](https://licensezero.com/licenses/relicense) sets out the terms of agreement between a contributor and a sponsor paying to relicense their contributions.  When a sponsor pays the price, [Artless Devices](#artless-devices) signs the relicense agreement with the sponsor on the contributor's behalf, as their agent.

The contributor's obligations are set out in the "Relicensing" section of the agreement.  Once the contributor receives payment, they're obligated to take the steps listed there.

## <a id="licensezero.com">licensezero.com</a>

licensezero.com is a website and API for selling [private licenses](#private-licenses), closing [relicense deals](#relicensing), and generating [waivers](#waivers).

You can think of licensezero.com as a kind of Internet vending machine.  As a contributor, you can make a deal with the operator of the vending machine to stock it with private licenses and relicense deals for sale.  The vending machine then handles taking payment and issuing licenses on your behalf.

You could also think of licensezero.com as a kind of e-commerce platform service-izing the back-office operations of a dual licensing business:negotiation, communication, payment processing, records management.  Rather than start a company and hire to handle those tasks, or take time away from development to do them yourself, you can use licensezero.com to run your licensing business.  It works just a well for expensive licenses priced in the thousands of dollars and relatively cheap licenses costing just a few bucks.

### <a id="stripe-connect">Stripe Connect</a>

You can create an account to sell private licenses through License Zero by linking a standard [Stripe](https://stripe.com) payment processing account, via [Stripe Connect](https://stripe.com/connect).  Stripe Connect enables License Zero to connect its Stripe account to yours, then generate access keys to initiate payment processing requests directly on your account.  Stripe assesses its fee, and any commission for licensezero.com, on the transaction.

### <a id="identifiers">Identifiers</a>

licensezero.com generates unique identifiers for each developer and each project.  The identifiers are version 4 [UUIDs](https://en.wikipedia.org/wiki/Universally_unique_identifier) like:

    daf5a8b1-23e0-4a9f-a6c1-69c40c71816b

### <a id="cryptography">Cryptography</a>

When you connect a Stripe account to licensezero.com, the site creates a unique UUID for you, as well as a [NaCl](https://nacl.cr.yp.to/)-style [Ed25519](https://ed25519.cr.yp.to/software.html) cryptographic signing keypair.  licensezero.com signs most kind of licensing information, from package metadata to `LICENSE` files, private licenses, waivers, and relicensing agreements, with the keypair created for your account, as well as its own keypair.

licensezero.com publishes the public signing key generated for you on your project pages, and via the API.  Users can leverage the [command line interface](#command-line-interface) to verify signatures on private licenses and other documents against the public key.

licensezero.com will not share the secret signing key generated for you with anyone, even with you.  But using the API, through the command line interface, you can have licensezero.com sign [waivers](#waivers) and package metadata on your behalf.

### <a id="project-pages">Project Pages</a>

licensezero.com serves a page with project information and purchase forms for each project.  For example:

<https://licensezero.com/projects/daf5a8b1-23e0-4a9f-a6c1-69c40c71816b>

The URL pattern is:

```text
https://licensezero.com/projects/{UUID}
```

Where `{UUID}` is the UUIDv4 for your project.

### <a id="pricing-graphics">Pricing Graphics</a>

licensezero.com serves SVG graphics with private-license pricing information that you can include in your project's `README` file or other documentation.  For example:

<figure><img alt="a pricing graphic" src="https://licensezero.com/projects/daf5a8b1-23e0-4a9f-a6c1-69c40c71816b/badge.svg"></figure>

The URL pattern is:

```text
https://licensezero.com/projects/{UUID}/badge.svg
```

Where `{UUID}` is the UUIDv4 for your project.

The Markdown syntax is:

```markdown
![L0](https://licensezero.com/projects/{UUID}/badge.svg)
```

## <a id="artless-devices">Artless Devices</a>

Artless Devices LLC is a California limited liability company.  It serves as the legal counterpart to those technical systems.  As a maintainer, you will have three legal relationships with Artless Devices:

### <a id="service-provider">Service Provider</a>

To use licensezero.com and its API, you must agree to [terms of service](https://licensezero.com/terms/service) with Artless Devices.  The terms were written to be read, and you should read them.  They set important rules about responsibility and liability, and make very explicit that Artless Devices isn't going to provide, or be responsible for, any legal advice about whether License Zero is right for you or your project.

### <a id="licensing-agent">Licensing Agent</a>

To offer private licenses for sale through licensezero.com, you agree to the [agency terms](https://licensezero.com/terms/agency) with Artless Devices.  You should definitely read that agreement, and again, it was written to be read, not to glaze your eyes over.  But there are a few especially important aspects, from Artless Devices' point of view.

First and foremost, you appoint Artless Devices as your agent to sign license agreements, waivers, and relicense agreements on your behalf.  That means that Artless Devices can do the deals you authorize through the API, via licensezero.com, with the same effect as if you'd signed them yourself.

Second, Artless Devices earns commission on private license sales.  The amount is set out in the agency terms when you offer the project for private licensing.

Third, you guarantee that you have the rights to offer the software you offer to license through License Zero, and to keep your project's metadata in line with what the [command line interface](#command-line-interface) generates.  The guarantee helps protect Artless Devices from those trying to take license fees for others' work.  Your promise about metadata helps ensure that the [command line interface](#command-line-interface) works as intended for users buying licenses.

Fourth, the relationship isn't exclusive, and either side can end it at any time.  You can sell private licenses through other services, or on your own.

Finally, neither the terms of service nor the agency terms change ownership of any intellectual property in your projects.  You own your rights, and you keep them.  Artless Devices doesn't receive a license from you and license users.  Rather, you license users and agree to relicense projects directly.  Artless Devices merely acts as your agent, signing on your behalf, as you've directed.

### <a id="toolmaker">Toolmaker</a>

Artless Devices owns and licenses the intellectual property in the [command line interface](#command-line-interface) that you will use to register and offer licenses for sale, as well as [forms and other software tools published online](https://github.com/licensezero).

## <a id="command-line-interface">Command Line Interface</a>

The [command line interface](https://www.npmjs.com/package/licensezero) is the primary way developers interact with licensezero.com.

With [Node.js](https://nodejs.org) and [npm](https://npmjs.com) installed, you can install the CLI with:

```bash
npm install --global licensezero
licensezero --help
```

You can also run the latest version of the `licensezero` command using `npx`:

```bash
npx licensezero --help
```

### <a id="identify-yourself">Identify Yourself</a>

To use the CLI as a contributor, user, or both, first run `licensezero identify`:

```bash
l0 identify "Jane Dev" US-CA jane@example.com
```

Provide your exact legal name, an [ISO 3166-2](https://en.wikipedia.org/wiki/ISO_3166-2) code for your tax jurisdiction, and your e-mail address.

A few example tax jurisdictions:

- `BR-SP` for São Paulo, Brazil
- `DE-HH` for Hamburg, Germany
- `GB-LND` for the City of London
- `JP-13` for Tokyo, Japan
- `NG-LA` for Lagos, Nigeria
- `US-TX` for Texas, United States

### As a Contributor <a id="as-a-contributor"></a>

#### <a id="registering">Registering</a>

To offer private licenses for sale via [licensezero.com](https://licensezero.com), you need to [identify yourself](#identify-yourself) with `licensezero identify`, then register as a licensor and connect a standard [Stripe](https://stripe.com) account:

```shell
l0 register
```

`l0 register` will open a page in your browser where you can log into Stripe, or create an account and connect it.  Once you've connected a Stripe account, [licensezero.com](https://licensezero.com) will provide you a licensor identifier and an access token that you can use to create a licensor profile:

```shell
l0 set-licensor-id $LICENSOR_ID
```

The command will then prompt for your access token, and save it for future use.

#### <a id="offering-private-licenses">Offering Private Licenses</a>

To offer private licenses for a project:

```shell
cd node-project
l0 offer --private 300
```

If you like, you can also set a [relicensing](#relicensing) price:

```bash
licensezero offer 50  --relicense 200000
```

You can run the `reprice` subcommand to update pricing.

Once you've registered the project, use the CLI to set licensing metadata and `LICENSE`:

```bash
licensezero license $PROJECT_ID --noncommercial
# or
licensezero license $PROJECT_ID --reciprocal
git add package.json LICENSE
git commit -m "License Zero"
git push
```

These commands will write a cryptographically signed `LICENSE` file, and plus `license` and `licensezero` properties in `package.json`.  The additions to `package.json` allow the CLI to identify the package for users quoting and buying private licenses.

#### <a id="locks">Locks</a>

By default, you can change pricing for private licenses at any time.  You could offer private licenses for $5 today, and $5,000 tomorrow.

In order to provide a publicly verifiable guarantee of license availability and pricing, either to users or others who want to [build on your work](#stacked-licensing), you can lock private-license pricing for your project to no more than the current price for a term of days, months, or even years.

```bash
licensezero lock $PROJECT_ID $UNLOCK_DATE
```

The unlock date must be a date at least seven calendar days in the future.

Locking a project prevents you from increasing pricing for your project or withdrawing your offer of private licenses.  For specifics, see [the agency terms](https://licensezero.com/terms/agency).

Please note that _locks are irrevocable_.  Artless Devices will not unlock a project early under any circumstances.

#### <a id="generating-waivers">Generating Waivers</a>

To generate a waiver for a project, provide a legal name, a jurisdiction code, and either a term in calendar days or `forever`:

```shell
l0 waive $PRODUCT_ID --name "Eve Able" --jurisdiction US-NY --term 90 > waiver.json
```

You can also issue waivers that don't expire:

```bash
l0 waive $PRODUCT_ID --name "Eve Able" --jurisdiction US-NY --days forever > waiver.json
```

#### <a id="retracting-projects">Retracting Projects</a>

You can stop offering private licenses through licensezero.com at any time:

```bash
licensezero retract $PROJECT_ID
```

Please note that under the [agency terms](https://licensezero.com/terms/agency), Artless Devices can complete private license and relicensing transactions that began before you retracted the project, but can't start new transactions.

### As a User <a id="as-a-user"></a>

### Quoting and Buying

You can generate quotes for License Zero software within the `node_modules` directories of your projects:

```shell
cd node-project
l0 quote
```

To buy missing licenses for dependencies of a project:

```shell
cd node-project
l0 buy
```

`l0 buy` will open a webpage in your browser listing the licenses to buy and taking credit card payment.  On successful purchase, [licensezero.com](https://licensezero.com) will provide the address of a purchase bundle that you can use to import all of the licenses you've just purchased at once:

```shell
l0 import-bundle https://licensezero.com/purchases/{code}
```

#### Importing Waivers

To import a waiver:

```shell
l0 import-waiver waiver.json
```

The command will open a page in your browser where you can log into Stripe or create an account, then link that account to licensezero.com.  Once you've successfully linked your account, you'll receive an identifier and an access token that you can save with:

#### Sponsoring Projects

To sponsor relicensing of a project onto permissive terms:

```shell
l0 sponsor $PROJECT_ID
```

The command will open a payment page in your web browser.

## <a id="ecosystems">Ecosystem Support</a>

Any open developer can choose one of the [public license options](#public-licenses), add its text to `LICENSE`, [offer private licenses through the API](#offering-private-liceness), and link to their [project page](#project-pages) in `README` or on a website.  Users can buy private licenses directly from the project page, and download their signed license file.  But the process becomes much simpler---much closer to zero friction, for both buyers and sellers---when the [command line interface](#command-line-interface) can find, read, and write package metadata showing which packages correspond to which licensezero.com project IDs.

Currently, the [command line interface](#command-line-interface) can comprehends metadata for [npm packages](https://www.npmjs.com).  But support for other ecosystems is planned.  If you're interested in expanding first-class tooling support to your ecosystem of choice, [reach out in the GitHub repository for the Go port of the CLI](https://github.com/licensezero/cli).  Even if you can't contribute code, pointers to documentation, examples of conventions, and answers to questions will be very helpful.

As of June of 2018, first-class support for Maven packages is next on the roadmap.

## <a id="contributions">Contributions</a>

As a independent software maintainer, you can license your work under both a [public license](#public-license) and [private licenses](#private-licenses) at the same time because you own the intellectual property in your work that others needs licenses for.  In other words, you can dual license your work because you own it.

When others contribute to your work, they will own the intellectual property in their contributions, not you.  As a result, users of your combined work will need licenses from you and from other contributors.  There are two straightforward ways to achieve that.

### <a id="parallel-licensing">Parallel Licensing</a>

You can choose to take contributions to your project only from those who license their contributions under permissive open source terms.  For example, you might license your work on a project under L0&#x2011;R, but ask contributors to license their work under [L0&#x2011;P](#permissive-license), and append the text of that license to your project's `LICENSE` file with a note that others' contributions come under that license.

Users of the combined project would then receive a license from you on the terms of L0&#x2011;R, for your contributions, and licenses from other contributors on the terms of L0&#x2011;P, for their contributions.  Would-be users who won't abide by the open source release conditions of your license can still buy a private license from you, for your contributions.  The private license for your work, plus the permissive license for others' contributions, cover all contributions.

In that kind of situation, you can sell private licenses for your contributions to the project, but others cannot.  Perhaps that feels completely fair.  If it doesn't, you may like to offer special credit, payment, or a free [waiver](#waiver) to contributors, to convince them to license their contributions under permissive terms.

### <a id="stacked-licensing">Stacked Licensing</a>

License Zero also supports projects that require multiple private licenses.  If you publish a project under L0&#x2011;R, and another developer forks the project, licensing their own work under L0&#x2011;R, too, they can append `package.json` metadata for a separate License Zero "project" in the same software.  Users who run the [command line interface](#command-line-interface) will see that they need a private license from each of you to use the project.  The same could happen with two contributors using L0&#x2011;NC, or contributors using a mix of License Zero licenses.

Note that as a contributor, you control pricing only for your own contributions, not anyone else's contributions, even if their work builds on yours.  Contributors building on top of work you license under L0&#x2011;NC will need to purchase private licenses from you to use and build on your work for the purpose of making money through License Zero, but otherwise, License Zero doesn't say anything about any relationship between you.

## <a id="complementary-approaches">Complementary Approaches</a>

License Zero isn't the only way to financially support your work on open software, and it tries its best to remain compatible with other opportunities.

- Ask people and companies for money, and hope they give it to you.  Set up a payment link in your `README` file.  Set yourself up on [Patreon](https://www.patreon.com).  Form a foundation and solicit donations.

- Sell books, stickers, shirts, and other physical goods with your project's name or logo.

- Sell or exchange the right to put company advertising, branding, or support acknowledgments in your project's documentation, on your project's website or social media accounts, or even in your software itself.

- Give paying customers early access to your work ahead of its release as open source, some time later.

- Require others to support you financially in order to use a trademark to identify themselves as providers of related products or services.  Trademark licensing for service providers is often coupled with advertising, in the form of a listing among service providers on the project's website.

- Charge companies, conferences, or other venues to train others in the use of your project, in person or remotely.

- Charge for access to and use of your time to assist users in using your work.

- Apply to grant-making institutions and government entities for financial support, if it aligns with their objectives.

- Charge others to install, configure, and run your project on their behalf.

- Charge others to integrate your work into their applications or services.

- Charge others for work they would like to do, such as bug fixes, feature adds, and other changes.

- Sell proprietary software relating to or enhancing your open project.  For example, many open source database companies license proprietary optimizations, monitoring tools, and replication features.
