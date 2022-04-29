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

### Important

Please, bear in mind this is not the place for new feature requests, ideas, or suggestions. This
place is only for important design, architectural decisions that are derived from already existing
feedback and experience, and as such, they have been already matured and discussed previously.


The proposal process
--------------------

We expect the Conan core development team to initiate most of them (other people starting them are
welcome, but notice this repository is for **substantial changes**. If you are not sure it's probably
better to [open an issue in the Conan repository](https://github.com/conan-io/conan/issues)). As
stated above, once those topics are mature after discussing them, they could be a part of a tribe
proposal in the future.

**A proposal starts with a pull-request to the `design` folder** of this repository
following the guidelines in this [template](design/_TEMPLATE.md).

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
| [@AndreasHK](https://github.com/AndreasHK) | [@jpovedano](https://github.com/jpovedano) | [@rdeterre](https://github.com/rdeterre) |
| [@barmaglot27](https://github.com/barmaglot27) | [@jsteinhofff](https://github.com/jsteinhofff) | [@rjktcby](https://github.com/rjktcby) |
| [@bjayanth](https://github.com/bjayanth) | [@Kellya-C](https://github.com/Kellya-C) | [@rockdreamer](https://github.com/rockdreamer) |
| [@bmanga](https://github.com/bmanga) | [@kenfred](https://github.com/kenfred) | [@saimusdev](https://github.com/saimusdev) |
| [@canmor](https://github.com/canmor) | [@KepptnKool](https://github.com/KepptnKool) | [@sourcedelica](https://github.com/sourcedelica) |
| [@climblinne](https://github.com/climblinne) | [@KerstinKeller](https://github.com/KerstinKeller) | [@SpoofedEx](https://github.com/SpoofedEx) |
| [@Da-LiFe](https://github.com/Da-LiFe) | [@kmaragon](https://github.com/kmaragon) | [@steinerthomas](https://github.com/steinerthomas) |
| [@Rycko1](https://github.com/Rycko1) | [@markushedvall](https://github.com/markushedvall) | [@sztomi](https://github.com/sztomi) |
| [@datalogics-kam](https://github.com/datalogics-kam) | [@madebr](https://github.com/madebr) | [@theodelrieu](https://github.com/theodelrieu) |
| [@davidtazy](https://github.com/davidtazy) | [@maikelvdh](https://github.com/maikelvdh) | [@Timen](https://github.com/Timen) |
| [@dean0x7d](https://github.com/dean0x7d) | [@mapau](https://github.com/mapau) | [@Tsubashi](https://github.com/Tsubashi) |
| [@detwiler](https://github.com/detwiler) | [@mathbunnyru](https://github.com/mathbunnyru) | [@uboot](https://github.com/uboot) |
| [@dheater](https://github.com/dheater) | [@michaelmaguire](https://github.com/michaelmaguire) | [@jason-d-walter](https://github.com/jason-d-walter) |
| [@DoDoENT](https://github.com/DoDoENT) | [@Minimonium](https://github.com/Minimonium) | [@wizardsd](https://github.com/wizardsd) |
| [@FabienLaurent](https://github.com/FabienLaurent) | [@monsdar](https://github.com/monsdar) | [@yipdw](https://github.com/yipdw) |
| [@foundry-markf](https://github.com/foundry-markf) | [@NewProggie](https://github.com/NewProggie) | [@ytimenkov](https://github.com/ytimenkov) |
| [@intelligide](https://github.com/intelligide) | [@ngerke](https://github.com/ngerke) | [@zacklj89](https://github.com/zacklj89) |
| [@ivzhh](https://github.com/ivzhh) | [@ohanar](https://github.com/ohanar) | [@brinkap](https://github.com/brinkap) |
| [@quazeeee](https://github.com/quazeeee) | [@tonka3000](https://github.com/tonka3000) | [@mrjoel](https://github.com/mrjoel) |
| [@Gernatch](https://github.com/Gernatch) | [@mpusz](https://github.com/mpusz) | [@ansutremel](https://github.com/ansutremel) |
| [@SpaceIm](https://github.com/SpaceIm) | [@gayanpathirage](https://github.com/gayanpathirage) | [@datalogics-robb](https://github.com/datalogics-robb)  |
| [@boussaffawalid](https://github.com/boussaffawalid) | [@sanblch](https://github.com/sanblch) | [@raplonu](https://github.com/raplonu)  |
| [@roalter](https://github.com/roalter) | [@sturmf](https://github.com/sturmf) | [@nguoithichkhampha](https://github.com/nguoithichkhampha) |
| [@Reg-Arvidson-Bose](https://github.com/Reg-Arvidson-Bose) | [@Glenn-Duffy-Bose](https://github.com/Glenn-Duffy-Bose) | [@FnGyula](https://github.com/FnGyula) |
| [@mmatrosov](https://github.com/mmatrosov) | [@setre14](https://github.com/setre14) | [@mohamedAlaaK](https://github.com/mohamedAlaaK) |


Special thanks to all of them for joining this effort.

Conan 2.0 is in alpha stage, it is approaching beta, and the Tribe is already large enough, so at the moment we are no longer accepting new members.
For any question, please send an email to [the Conan team](mailto:tribe-maintainers@conan.io?subject=Conan%20Tribe%20Question).
