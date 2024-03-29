**Description**

Author: Will Hong

It seems that another encrypted message has been intercepted. The encryptor seems to have learned their lesson though and now there isn't any punctuation! Can you still crack the cipher? Download the message here.

Hints
1) Try refining your frequency attack, maybe analyzing groups of letters would improve your results?

**Solve**

This challenge is very similar to the [past challenges](./crypto/substitution0(100).md) in this series of challenges, following that guide will help before starting this challenge.

Ciphered:
```
cwaiaafqzczakailbmcwaijabbazclnbqzwaxwqowzswmmbsmtdrcaizasriqcusmtdacqcqmezqesbrxqeosunaidlciqmclexrzsunaiswlbbaeoacwazasmtdacqcqmezpmsrzdiqtliqbumezuzcatzlxtqeqzcilcqmeprexltaeclbzjwqswliakaiurzaprblextlihaclnbazhqbbzwmjakaijanabqakacwadimdaidridmzamplwqowzswmmbsmtdrcaizasriqcusmtdacqcqmeqzemcmebucmcalswklbrlnbazhqbbznrclbzmcmoaczcrxaeczqecaiazcaxqelexafsqcaxlnmrcsmtdrcaizsqaesaxapaezqkasmtdacqcqmezliampcaeblnmiqmrzlpplqizlexsmtaxmjecmireeqeoswashbqzczlexafasrcqeosmepqozsiqdczmppaezamecwamcwaiwlexqzwalkqbupmsrzaxmeafdbmilcqmelexqtdimkqzlcqmelexmpcaewlzabataeczmpdblujanabqakalsmtdacqcqmecmrswqeomecwamppaezqkaabataeczmpsmtdrcaizasriqcuqzcwaiapmialnaccaikawqsbapmicaswakleoabqztcmzcrxaeczqeltaiqslewqowzswmmbzpricwaijanabqakacwlclerexaizclexqeompmppaezqkacasweqyrazqzazzaecqlbpmitmrecqeoleappascqkaxapaezalexcwlccwacmmbzlexsmepqorilcqmepmsrzaesmrecaiaxqexapaezqkasmtdacqcqmezxmazemcbalxzcrxaeczcmhemjcwaqiaeatulzappascqkabulzcalswqeocwatcmlscqkabucwqehbqhalelcclshaidqsmscpqzlemppaezqkabumiqaecaxwqowzswmmbsmtdrcaizasriqcusmtdacqcqmecwlczaahzcmoaeailcaqecaiazcqesmtdrcaizsqaesaltmeowqowzswmmbaizcalswqeocwataemrowlnmrcsmtdrcaizasriqcucmdqyracwaqisriqmzqcutmcqklcqeocwatcmafdbmiamecwaqimjelexaelnbqeocwatcmnaccaixapaexcwaqitlswqeazcwapbloqzdqsmSCP{E6I4T_4E41U515_15_73X10R5_6SXLAL76}
```
Using the tool given in the writeup is strong enough to break the substitution for substitution2, using that will return the flag.

Unciphered:
```
thereexistseveralotherwellestablishedhighschoolcomputersecuritycompetitionsincludingcyberpatriotanduscyberchallengethesecompetitionsfocusprimarilyonsystemsadministrationfundamentalswhichareveryusefulandmarketableskillshoweverwebelievetheproperpurposeofahighschoolcomputersecuritycompetitionisnotonlytoteachvaluableskillsbutalsotogetstudentsinterestedinandexcitedaboutcomputersciencedefensivecompetitionsareoftenlaboriousaffairsandcomedowntorunningchecklistsandexecutingconfigscriptsoffenseontheotherhandisheavilyfocusedonexplorationandimprovisationandoftenhaselementsofplaywebelieveacompetitiontouchingontheoffensiveelementsofcomputersecurityisthereforeabettervehiclefortechevangelismtostudentsinamericanhighschoolsfurtherwebelievethatanunderstandingofoffensivetechniquesisessentialformountinganeffectivedefenseandthatthetoolsandconfigurationfocusencounteredindefensivecompetitionsdoesnotleadstudentstoknowtheirenemyaseffectivelyasteachingthemtoactivelythinklikeanattackerpicoctfisanoffensivelyorientedhighschoolcomputersecuritycompetitionthatseekstogenerateinterestincomputerscienceamonghighschoolersteachingthemenoughaboutcomputersecuritytopiquetheircuriositymotivatingthemtoexploreontheirownandenablingthemtobetterdefendtheirmachinestheflagispicoCTF{N6R4M_4N41Y515_15_73D10U5_6CDAEA76}
```

There is the flag!
It is: picoCTF{N6R4M_4N41Y515_15_73D10U5_6CDAEA76}
