# Conan 2.0 Tribe

During
recent years Conan has been successfully adopted by many individual programmers as well as  
enterprise teams, and helped them all to build their applications more easily and improve their 
development experience. The Conan team continues to identify new features and areas of improvement
based on everything we've learned in recent years.

This repository in Github will gather the proposals, substantial features and 
breaking changes.  For example, there are several such proposals being evaluated for Conan 2.0, and even after it's release, this repository will continue to be used in the same way for future releases. This aims to be a collaborative
effort among the tribe members and the development team, trying to understand 
the needs, the pains and the implications of every feature or breaking change.

We need to evolve Conan, but we want to do it together.


> Notice this is not the place for new feature requests, ideas or suggestions. 
  This is the place only for important design, architectural decisions that 
  are derived from already existing feedback and experience, and as such, they 
  have been already matured and discussed previously.


The proposal process
--------------------

**A proposal starts with a pull-request to the `design` folder** of this repository
following the guidelines in this [template](design/_TEMPLATE.md). We expect Conan
core development team to initiate most of them (other people starting them are
welcome, but notice this repository is for substantial changes. If you are not sure
it's probably better to 
[open an issue in Conan repository](https://github.com/conan-io/conan/issues)).

This is the lifecycle of a proposal:

 * Fork this repo and create a pull-request with your proposal following the
   [template in the design folder](design/_TEMPLATE.md). Name the file 
   `###-rfc-title.md` (do not assign a number yet).

   If this is affecting only one part of the existing proposal, open the pull-request
   modifying that file.

 * The tribe will be notified about the new proposal or modification.

 * Discussion will be held in the pull-request itself. Use Github provided features
   to share your comments and concerns (suggestions, pull-requests reviews, 
   reactions,...), or even open a pull-request to the origin branch to propose
   enhancements over existing commits.

 * Attention should focus on existing proposals before moving to new ones. We expect
   to move forward each proposal after one week if there is enough feedback, being it
   approved or rejected.

Ready? Have a look to existing ones [here](design/) or to 
[current ones being considered](https://github.com/conan-io/tribe/pulls).


Tribe members
-------------

These are the people that belong to the Conan 2.0 tribe. Their votes and comments
will receive special attention:


|                      |                     |                     |
|----------------------|---------------------|---------------------|
| [@a4z](https://github.com/a4z) | [@jamesweir-tomtom](https://github.com/jamesweir-tomtom) | [@p-groarke](https://github.com/p-groarke) |
| [@akleber](https://github.com/akleber) | [@jaredkeithwhite](https://github.com/jaredkeithwhite) | [@prince-chrismc](https://github.com/prince-chrismc) |
| [@albaltimore](https://github.com/albaltimore) | [@jasal82](https://github.com/jasal82) | [@puetzk](https://github.com/puetzk) |
| [@aleksa2808](https://github.com/aleksa2808) | [@jcar87](https://github.com/jcar87) | [@raplonu](https://github.com/raplonu) |
| [@AlexandrePTJ](https://github.com/AlexandrePTJ) | [@jmarrec](https://github.com/jmarrec) | [@rasjani](https://github.com/rasjani) |
| [@alexFickle](https://github.com/alexFickle) | [@jonatin](https://github.com/jonatin) | [@rconde01](https://github.com/rconde01) |
| [@AndreasHK](https://github.com/AndreasHK) | [@Jpovedano](https://github.com/Jpovedano) | [@rdeterre](https://github.com/rdeterre) |
| [@barmaglot27](https://github.com/barmaglot27) | [@jsteinhofff](https://github.com/jsteinhofff) | [@rjktcby](https://github.com/rjktcby) |
| [@bjayanth](https://github.com/bjayanth) | [@Kellya-C](https://github.com/Kellya-C) | [@rockdreamer](https://github.com/rockdreamer) |
| [@bmanga](https://github.com/bmanga) | [@kenfred](https://github.com/kenfred) | [@saimusdev](https://github.com/saimusdev) |
| [@canmor](https://github.com/canmor) | [@KepptnKool](https://github.com/KepptnKool) | [@sourcedelica](https://github.com/sourcedelica) |
| [@climblinne](https://github.com/climblinne) | [@KerstinKeller](https://github.com/KerstinKeller) | [@SpoofedEx](https://github.com/SpoofedEx) |
| [@Da-LiFe](https://github.com/Da-LiFe) | [@kmaragon](https://github.com/kmaragon) | [@steinerthomas](https://github.com/steinerthomas) |
| [@Daniel-Roberts-Bose](https://github.com/Daniel-Roberts-Bose) | [@mackanhedvall](https://github.com/mackanhedvall) | [@sztomi](https://github.com/sztomi) |
| [@datalogics-kam](https://github.com/datalogics-kam) | [@madebr](https://github.com/madebr) | [@theodelrieu](https://github.com/theodelrieu) |
| [@davidtazy](https://github.com/davidtazy) | [@maikelvdh](https://github.com/maikelvdh) | [@Timen](https://github.com/Timen) |
| [@dean0x7d](https://github.com/dean0x7d) | [@mapau](https://github.com/mapau) | [@Tsubashi](https://github.com/Tsubashi) |
| [@detwiler](https://github.com/detwiler) | [@mathbunnyru](https://github.com/mathbunnyru) | [@uboot](https://github.com/uboot) |
| [@dheater](https://github.com/dheater) | [@michaelmaguire](https://github.com/michaelmaguire) | [@walterj-adsk](https://github.com/walterj-adsk) |
| [@DoDoENT](https://github.com/DoDoENT) | [@Minimonium](https://github.com/Minimonium) | [@wizardsd](https://github.com/wizardsd) |
| [@FabienLaurent](https://github.com/FabienLaurent) | [@monsdar](https://github.com/monsdar) | [@yipdw](https://github.com/yipdw) |
| [@foundry-markf](https://github.com/foundry-markf) | [@NewProggie](https://github.com/NewProggie) | [@ytimenkov](https://github.com/ytimenkov) |
| [@intelligide](https://github.com/intelligide) | [@ngerke](https://github.com/ngerke) | [@zacklj89](https://github.com/zacklj89) |
| [@ivzhh](https://github.com/ivzhh) | [@ohanar](https://github.com/ohanar) |  |

Special thanks to all of them for joining this effort.

To inquire about the member application process, please send an email to [the Conan team](mailto:info@conan.io?subject=Conan%20Tribe%20Question).
