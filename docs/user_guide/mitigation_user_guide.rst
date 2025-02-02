.. _bias mitigation:

Bias Mitigation (Debias)
========================

The following guide is designed to present the more general details on 
using the package to mitigate (debias) bias in word embedding models. 
The following sections show:

- run :class:`~wefe.debias.hard_debias.HardDebias` mitigation method on an
  embedding model to mitigate gender bias (using the ``fit-transform`` interface).
- apply the ``target`` parameter when executing the transformation.
- apply the ``ignore`` parameter when executing the transformation.
- apply the ``copy`` parameter when executing the transformation.
- run :class:`~wefe.debias.multiclass_hard_debias.MulticlassHardDebias` mitigation 
  method on an word embedding model to mitigate ethnic bias.

.. note::

    For a list of metrics implemented in WEFE, refer to the
    :ref:`debias-API` of the API reference.  

.. note::

    If you want to know more about WEFE's standardization of debias methods, 
    visit :ref:`mitigation framework` in the conceptual guides.


Hard Debias
-----------


Hard debias is a method that allows mitigating biases through geometric operations 
on embeddings. 
This method is binary because it only allows 2 classes of the same bias criterion,
such as male or female.

.. note::

    For a multiclass debias (such as for Latinos, Asians and Whites), it is
    recommended to visit
    :class:`~wefe.debias.multiclass_hard_debias.MulticlassHardDebias` class.
    

The main idea of this method is:

1. Identify a bias subspace through the defining sets. In the case of gender,
these could be e.g. ``[['woman', 'man'], ['she', 'he'], ...]``

2. Neutralize the bias subspace of embeddings that should not be biased.

First, we define a set of words that are correct to be related to the bias
criterion: the *criterion specific gender words*.
For example, in the case of gender, *gender specific* words are:
``['he', 'his', 'He', 'her', 'she', 'him', 'him', 'She', 'man', 'women', 'men'...]``.

We then define that all words outside this set should have no relation to the
bias criterion and thus have the possibility of being biased. (e.g. for the case of
genthe bias direction, such that neither is closer to the bias direction
than the other: ``['doctor', 'nurse', ...]``). Therefore, this set of words is
neutralized with respect to the bias subspace found in the previous step.

The neutralization is carried out under the following operation:

- :math:`u` : embedding
- :math:`v`: bias direction

First calculate the projection of the embedding on the bias subspace.

.. math::

    \text{bias subspace} = \frac{v \cdot (v \cdot u)}{(v \cdot v)}

Then subtract the projection from the embedding.

.. math::

    u' = u - \text{bias subspace}

3. Equalizate the embeddings with respect to the bias direction.

Given an equalization set (set of word pairs such as ``['she', 'he'],
['men', 'women'], ...``, but not limited to the definitional set) this step
executes, for each pair, an equalization with respect to the bias direction.
That is, it takes both embeddings of the pair and distributes them at the same
distance from the bias direction, so that neither is closer to the bias direction
than the other.


The fit parameters define how the neutralization will be calculated. In
Hard Debias, you have to provide the the ``definitional_pairs``, the
``equalize_pairs`` (which could be the same of definitional pairs) and
optionally, a debias ``criterion_name`` (to name the debiased model).


The code shown below shows how to run Hard Debias from gender to the test model 
provided by wefe (reduced word2vec).

.. code:: ipython3

    from wefe.utils import load_test_model
    
    model = load_test_model()  # load a reduced version of word2vec
    model




.. parsed-literal::

    <wefe.word_embedding_model.WordEmbeddingModel at 0x7f3b83b3b700>



Load the required word sets.

.. code:: ipython3

    from wefe.datasets import fetch_debiaswe
    from wefe.debias.hard_debias import HardDebias
    
    debiaswe_wordsets = fetch_debiaswe()
    
    definitional_pairs = debiaswe_wordsets["definitional_pairs"]
    equalize_pairs = debiaswe_wordsets["equalize_pairs"]
    gender_specific = debiaswe_wordsets["gender_specific"]
    
    
    print(f"definitional_pairs: \n{definitional_pairs}")
    print(f"equalize_pairs: \n{equalize_pairs}")
    print(f"gender_specific: \n{gender_specific}")
    print("-" * 70, "\n")


.. parsed-literal::

    definitional_pairs: 
    [['woman', 'man'], ['girl', 'boy'], ['she', 'he'], ['mother', 'father'], ['daughter', 'son'], ['gal', 'guy'], ['female', 'male'], ['her', 'his'], ['herself', 'himself'], ['Mary', 'John']]
    equalize_pairs: 
    [['monastery', 'convent'], ['spokesman', 'spokeswoman'], ['Catholic_priest', 'nun'], ['Dad', 'Mom'], ['Men', 'Women'], ['councilman', 'councilwoman'], ['grandpa', 'grandma'], ['grandsons', 'granddaughters'], ['prostate_cancer', 'ovarian_cancer'], ['testosterone', 'estrogen'], ['uncle', 'aunt'], ['wives', 'husbands'], ['Father', 'Mother'], ['Grandpa', 'Grandma'], ['He', 'She'], ['boy', 'girl'], ['boys', 'girls'], ['brother', 'sister'], ['brothers', 'sisters'], ['businessman', 'businesswoman'], ['chairman', 'chairwoman'], ['colt', 'filly'], ['congressman', 'congresswoman'], ['dad', 'mom'], ['dads', 'moms'], ['dudes', 'gals'], ['ex_girlfriend', 'ex_boyfriend'], ['father', 'mother'], ['fatherhood', 'motherhood'], ['fathers', 'mothers'], ['fella', 'granny'], ['fraternity', 'sorority'], ['gelding', 'mare'], ['gentleman', 'lady'], ['gentlemen', 'ladies'], ['grandfather', 'grandmother'], ['grandson', 'granddaughter'], ['he', 'she'], ['himself', 'herself'], ['his', 'her'], ['king', 'queen'], ['kings', 'queens'], ['male', 'female'], ['males', 'females'], ['man', 'woman'], ['men', 'women'], ['nephew', 'niece'], ['prince', 'princess'], ['schoolboy', 'schoolgirl'], ['son', 'daughter'], ['sons', 'daughters'], ['twin_brother', 'twin_sister']]
    gender_specific: 
    ['he', 'his', 'He', 'her', 'she', 'him', 'She', 'man', 'women', 'men', 'His', 'woman', 'spokesman', 'wife', 'himself', 'son', 'mother', 'father', 'chairman', 'daughter', 'husband', 'guy', 'girls', 'girl', 'Her', 'boy', 'King', 'boys', 'brother', 'Chairman', 'spokeswoman', 'female', 'sister', 'Women', 'Man', 'male', 'herself', 'Lions', 'Lady', 'brothers', 'dad', 'actress', 'mom', 'sons', 'girlfriend', 'Kings', 'Men', 'daughters', 'Prince', 'Queen', 'teenager', 'lady', 'Bulls', 'boyfriend', 'sisters', 'Colts', 'mothers', 'Sir', 'king', 'businessman', 'Boys', 'grandmother', 'grandfather', 'deer', 'cousin', 'Woman', 'ladies', 'Girls', 'Father', 'uncle', 'PA', 'Boy', 'Councilman', 'mum', 'Brothers', 'MA', 'males', 'Girl', 'Mom', 'Guy', 'Queens', 'congressman', 'Dad', 'Mother', 'grandson', 'twins', 'bull', 'queen', 'businessmen', 'wives', 'widow', 'nephew', 'bride', 'females', 'aunt', 'Congressman', 'prostate_cancer', 'lesbian', 'chairwoman', 'fathers', 'Son', 'moms', 'Ladies', 'maiden', 'granddaughter', 'younger_brother', 'Princess', 'Guys', 'lads', 'Ma', 'Sons', 'lion', 'Bachelor', 'gentleman', 'fraternity', 'bachelor', 'niece', 'Lion', 'Sister', 'bulls', 'husbands', 'prince', 'colt', 'salesman', 'Bull', 'Sisters', 'hers', 'dude', 'Spokesman', 'beard', 'filly', 'Actress', 'Him', 'princess', 'Brother', 'lesbians', 'councilman', 'actresses', 'Viagra', 'gentlemen', 'stepfather', 'Deer', 'monks', 'Beard', 'Uncle', 'ex_girlfriend', 'lad', 'sperm', 'Daddy', 'testosterone', 'MAN', 'Female', 'nephews', 'maid', 'daddy', 'mare', 'fiance', 'Wife', 'fiancee', 'kings', 'dads', 'waitress', 'Male', 'maternal', 'heroine', 'feminist', 'Mama', 'nieces', 'girlfriends', 'Councilwoman', 'sir', 'stud', 'Mothers', 'mistress', 'lions', 'estranged_wife', 'womb', 'Brotherhood', 'Statesman', 'grandma', 'maternity', 'estrogen', 'ex_boyfriend', 'widows', 'gelding', 'diva', 'teenage_girls', 'nuns', 'Daughter', 'czar', 'ovarian_cancer', 'HE', 'Monk', 'countrymen', 'Grandma', 'teenage_girl', 'penis', 'bloke', 'nun', 'Husband', 'brides', 'housewife', 'spokesmen', 'suitors', 'menopause', 'monastery', 'patriarch', 'Beau', 'motherhood', 'brethren', 'stepmother', 'Dude', 'prostate', 'Moms', 'hostess', 'twin_brother', 'Colt', 'schoolboy', 'eldest', 'brotherhood', 'Godfather', 'fillies', 'stepson', 'congresswoman', 'Chairwoman', 'Daughters', 'uncles', 'witch', 'Mommy', 'monk', 'viagra', 'paternity', 'suitor', 'chick', 'Pa', 'fiancé', 'sorority', 'macho', 'Spokeswoman', 'businesswoman', 'eldest_son', 'gal', 'statesman', 'schoolgirl', 'fathered', 'goddess', 'hubby', 'mares', 'stepdaughter', 'blokes', 'dudes', 'socialite', 'strongman', 'Witch', 'fiancée', 'uterus', 'grandsons', 'Bride', 'studs', 'mama', 'Aunt', 'godfather', 'hens', 'hen', 'mommy', 'Babe', 'estranged_husband', 'Fathers', 'elder_brother', 'boyhood', 'baritone', 'Diva', 'Lesbian', 'grandmothers', 'grandpa', 'boyfriends', 'feminism', 'countryman', 'stallion', 'heiress', 'queens', 'Grandpa', 'witches', 'aunts', 'semen', 'fella', 'granddaughters', 'chap', 'knight', 'widower', 'Maiden', 'salesmen', 'convent', 'KING', 'vagina', 'beau', 'babe', 'HIS', 'beards', 'handyman', 'twin_sister', 'maids', 'gals', 'housewives', 'Gentlemen', 'horsemen', 'Businessman', 'obstetrics', 'fatherhood', 'beauty_queen', 'councilwoman', 'princes', 'matriarch', 'colts', 'manly', 'ma', 'fraternities', 'Spokesmen', 'pa', 'fellas', 'Gentleman', 'councilmen', 'dowry', 'barbershop', 'Monks', 'WOMAN', 'fraternal', 'ballerina', 'manhood', 'Dads', 'heroines', 'granny', 'gynecologist', 'princesses', 'Goddess', 'yo', 'Granny', 'knights', 'eldest_daughter', 'HER', 'underage_girls', 'masculinity', 'Girlfriend', 'bro', 'Grandmother', 'grandfathers', 'crown_prince', 'Restless', 'paternal', 'Queen_Mother', 'Boyfriend', 'womens', 'Males', 'SHE', 'Countess', 'stepchildren', 'Belles', 'bachelors', 'matron', 'momma', 'Legs', 'maidens', 'goddesses', 'landlady', 'sisterhood', 'Grandfather', 'Fraternity', 'Majesty', 'Babes', 'lass', 'maternal_grandmother', 'blondes', "ma'am", 'Womens', 'divorcee', 'Momma', 'fathering', 'Effie', 'Lad', 'womanhood', 'missus', 'Sisterhood', 'granddad', 'Mens', 'papa', 'gf', 'sis', 'Husbands', 'Hen', 'womanizer', 'gynecological', 'stepsister', 'Handsome', 'Prince_Charming', 'BOY', 'stepdad', 'teen_ager', 'GIRL', 'dame', 'Sorority', 'beauty_pageants', 'raspy', 'harem', 'maternal_grandfather', 'Hes', 'deliveryman', 'septuagenarian', 'damsel', 'paternal_grandmother', 'paramour', 'paternal_grandparents', 'Nun', 'DAD', 'mothering', 'shes', "HE_'S", 'Nuns', 'teenage_daughters', 'auntie', 'widowed_mother', 'Girlfriends', 'FATHER', 'virile', 'COUPLE', 'grandmas', 'Hubby', 'nan', 'vixen', 'Joan_Crawford', 'stepdaughters', 'endometrial_cancer', 'stepsons', 'loins', 'Grandson', 'Mitchells', 'erections', 'Matron', 'Fella', 'daddies', 'ter', 'Sweetie', 'Dudes', 'Princesses', 'Lads', 'lioness', 'Mamma', 'virility', 'bros', 'womenfolk', 'Heir', 'BROTHERS', 'manliness', 'patriarchs', 'earl', 'sisterly', 'Whore', 'Gynaecology', 'countess', 'convents', 'Oratory', 'witch_doctor', 'mamas', 'yah', 'aunty', 'aunties', 'Heiress', 'lasses', 'Breasts', 'fairer_sex', 'sorority_sisters', 'WIFE', 'Laurels', 'penile', 'nuh', 'mah', 'toms', 'mam', 'Granddad', 'premenopausal_women', 'Granddaddy', 'nana', 'coeds', 'dames', 'herdsman', 'Mammy', 'Fellas', 'Niece', 'menfolk', 'Grandad', 'bloods', 'Gramps', 'damsels', 'Granddaughter', 'mamma', 'concubine', 'Oros', 'Blarney', 'filial', 'broads', 'Ethel_Kennedy', 'ACTRESS', 'Tit', 'fianc', 'Hunk', 'Night_Shift', 'wifey', 'Lothario', 'Holy_Roman_Emperor', 'horse_breeder', 'grandnephew', 'Lewises', 'Muscular', 'feminist_movement', 'Sanan', 'womenâ_€_™', 'Fiancee', 'dowries', 'Carmelite', 'rah', 'n_roller', 'bay_filly', 'belles', 'Uncles', 'PRINCESS', 'womans', 'Homeboy', 'Blokes', 'Charmer', 'codger', 'Delta_Zeta', 'courtesans', 'grandaughter', 'SISTER', 'Highness', 'grandbabies', 'crone', 'Skip_Away', 'noblewoman', 'bf', 'jane', 'philandering_husband', 'Sisqo', 'mammy', 'daugher', 'director_Skip_Bertman', 'DAUGHTER', 'Royal_Highness', 'mannish', 'spinsters', 'Missus', 'madame', 'Godfathers', 'saleswomen', 'beaus', 'Risha', 'luh', 'sah', 'negligee', 'Womenâ_€_™', 'Hos', 'salesgirl', 'grandmom', 'Grandmas', 'Lawsons', 'countrywomen', 'Booby', 'darlin', 'Sheiks', 'boyz', 'wifes', 'Bayi', 'Il_Duce', 'â_€_œMy', 'fem', 'daugther', 'Potti', 'hussy', 'tch', 'Gelding', 'stemmed_roses', 'Damson', 'puh', 'Tylers', 'neice', 'Mutha', 'GRANDMOTHER', 'youse', 'spurned_lover', 'mae', 'Britt_Ekland', 'clotheshorse', 'Carlita_Kilpatrick', 'Cambest', 'Pretty_Polly', 'banshees', 'male_chauvinist', 'Arliss', 'mommas', 'maidservant', 'Gale_Harold', 'Little_Bo_Peep', 'Cleavers', 'hags', 'blowsy', 'Queen_Elizabeth_I.', 'lassies', 'papas', 'BABE', 'ugly_ducklings', 'Jims', 'hellion', 'Beautician', 'coalminer', 'relaxin', 'El_Mahroug', 'Victoria_Secret_Angel', 'shepherdess', 'Mosco', 'Slacks', 'nanna', 'wifely', 'tomboys', 'LAH', 'hast', 'apo', 'Kaplans', 'milkmaid', 'Robin_Munis', 'John_Barleycorn', 'royal_highness', 'Meanie', 'NAH', 'trollop', 'roh', 'Jewess', 'Sheik_Hamad', 'mumsy', 'Big_Pussy', 'chil_dren', 'Aunt_Bea', 'basso', 'sista', 'girlies', 'nun_Sister', 'chica', 'Bubbas', 'massa', 'Southern_belles', 'Nephews', 'castrations', 'Mister_Ed', 'Grandsons', 'Calaf', 'Malachy_McCourt', 'Shamash', 'hey_hey', 'Harmen', 'sonofabitch', 'Donovans', 'Grannie', 'Kalinka', 'hisself', 'Devean', 'goatherd', 'hinds', 'El_Corredor', 'Kens', 'notorious_womanizer', 'goh', 'Mommas', 'washerwoman', 'Samaira', 'Coo_Coo', 'Governess', 'grandsire', 'PRINCE_WILLIAM', 'gramma', 'him.He', 'Coptic_priest', 'Corbie', 'Kennys', 'thathe', 'Pa_Pa', 'Bristols', 'Hotep', 'snowy_haired', 'El_Prado_Ire', 'Girl_hitmaker', 'Hurleys', 'St._Meinrad', 'sexually_perverted', 'authoress', 'Prudie', 'raven_haired_beauty', 'Bonos', 'domestic_shorthair', 'brothas', 'nymphet', 'Neelma', 'Seita', 'stud_muffin', 'St._Judes', 'yenta', 'bare_shouldered', 'Pinkney_Sr.', 'PRINCE_CHARLES', 'Bisutti', 'sistas', 'Blanche_Devereaux', 'Momoa', 'Quiff', 'Scotswoman', 'balaclava_clad_men', 'Louis_Leakey', 'dearie', 'vacuum_cleaner_salesman', 'grandads', 'postulant', 'SARAH_JESSICA_PARKER', 'AUNT', 'Prince_Dauntless', 'Dalys', 'Darkie', 'Czar_Nicholas', 'Lion_Hearted', 'Boy_recliner', 'baby_mamas', 'giantess', 'Lawd', 'GRANNY', 'fianc_e', 'Bilqis', 'WCTU', 'famly', 'Ellas', 'feminazis', 'Pentheus', 'MAMAS', 'Town_Criers', 'Saggy', 'youngman', 'grandam', 'divorcé', 'bosomed', 'roon', 'Simmentals', 'eponymous_heroine', 'LEYLAND', "REE'", "cain't", 'Evelynn', "WAH'", 'sistah', 'Horners', 'Elsie_Poncher', 'Coochie', 'rat_terriers', 'Limousins', 'Buchinski', 'Schicchi', 'Carpitcher', 'Khwezi', "HAH'", 'Shazza', 'Mackeson', "ROH'", 'kuya', 'novice_nun', 'Shei', 'Elmasri', 'ladykiller', '6yo', 'Yenta', 'SHEL', 'pater', 'Souse', 'Tahirah', 'comedian_Rodney_Dangerfield', 'Shottle', 'carryin', 'Sath', "fa'afafine", 'royal_consort', 'hus_band', 'maternal_uncles', 'dressing_provocatively', 'dreamgirl', 'millionaire_industrialist', 'Georgie_Girl', 'Must_Be_Obeyed', 'joh', 'Arabian_stallion', 'ahr', 'mso_para_margin_0in', "SOO'", 'Biddles', 'Chincoteague_Volunteer_Fire', 'Lisa_Miceli', 'gorgeous_brunette', 'fiancŽ', 'Moved_fluently', 'Afternoon_Deelites', 'biker_dude', 'Vito_Spatafore', 'MICK_JAGGER', 'Adesida', 'Reineman', 'witz', 'Djamila', 'Glenroe', 'daddys', 'Romanzi', 'gentlewomen', 'Dandie_Dinmont_terrier', 'Excess_Ire', 'By_SYVJ_Staff', 'zan', 'CONFESSIONS', 'Magees', 'wimmin', 'tash', 'Theatrical_Ire', 'Prince_Charmings', 'chocolate_eclair', 'bron', 'daughers', 'Felly', 'fiftyish', 'Spritely', 'GRANDPA', 'distaffer', 'Norbertines', "DAH'", 'leader_Muammar_Gadaffi', 'swains', 'Prince_Tomohito', 'Honneur', 'Soeur', 'jouster', 'Pharaoh_Amenhotep_III', 'QUEEN_ELIZABETH_II', "Ne'er", 'Galileo_Ire', 'Fools_Crow', 'Lannisters', 'Devines', 'gonzales', 'columnist_Ann_Landers', 'Moseleys', 'hiz', 'busch', 'roastee', 'toyboys', 'Sheffields', 'grandaunt', 'Galvins', 'Giongo', 'geh', 'flame_haired_actress', 'Grammarian', 'Greg_Evigan', 'frontierswoman', 'Debele', 'rabs', 'nymphets', 'aai', 'BREE', 'Shaqs', 'ZAY', 'pappa', 'Housa', 'refrigerator_repairman', 'artificial_inseminations', 'chickie', 'Rippa', 'teenager_Tracy_Turnblad', 'homebred_colt', 'Abigaille', 'hen_pecked_husband', 'businesman', 'her.She', 'Kaikeyi', 'Stittsworth', 'self_proclaimed_redneck', 'Khella', 'NeW', 'Evers_Swindell', 'Asmerom_Gebreselassie', 'Boy_recliners', 'Cliff_Claven', 'Legge_Bourke', 'Costos', "d'_honneur", 'sistahs', 'Cabble', 'sahn', 'CROW_AGENCY_Mont', 'jezebel', 'Harrolds', 'ROSARIO_DAWSON', 'INXS_frontman_Michael_Hutchence', 'Gursikh', 'Dadas', 'VIAGA', 'keen_horsewoman', 'Theodoric', 'Eldery', 'lihn', 'Alice_Kramden', 'Santarina', 'radical_cleric_al_Sadr', 'Curleys', "SY'", 'Fidaa', 'Saptapadi', 'Actor_Sean_Astin', 'Kellita_Smith', 'Doly', 'Libertina', 'Money_McBags', 'Chief_Bearhart', 'choirgirl', 'chestnut_stallion', 'VIGRA', 'BY_JIM_McCONNELL', 'Sal_Vitale', 'Trivia_buffs', 'kumaris', 'fraternal_lodge', 'galpals', 'Borino_Quinn', 'lina', 'LATEST_Rapper', 'Bezar', 'Manro', 'bakla', 'Grisetti', 'blond_bimbo', 'spinster_aunt', 'gurls', 'hiswife', 'paleface', 'Charlye', 'hippie_chicks', 'Khalifas', 'Picture_JUSTIN_SANSON', 'Hepburns', 'yez', 'ALDER', 'Sanussi', 'Lil_Sis', 'McLoughlins', 'Barbra_Jean', 'Lulua', 'thatshe', 'actress_Shohreh_Aghdashloo', 'SIR_ANTHONY_HOPKINS', 'Gloddy', "ZAH'", "ORANGE_'S", 'Danielle_Bimber', 'grandmum', 'Kulkis', 'Brazington', 'Marisa_Lenhard_CFA', 'SIR_JOHN', 'Clareman', 'Aqila', 'Heavily_tattooed', 'Libbys', 'thim', 'elocutionist', 'submissives', 'Inja', 'rahm', 'Agnes_Gooch', 'fake_tits', 'nancy_boys', 'Swaidan', "SHAH'", "ain'ta_bed", 'Shumail_Raj', 'Duchesse', 'diethylstilbestrol_DES', 'colt_foal', 'unfaithful_lover', 'Maseri', 'nevah', 'SAHN', 'Barths', 'Toughkenamon', 'GUEST_STARS', 'him.But', 'Donna_Claspell', 'gingham_dresses', 'Massage_Parlour', 'wae', 'Wasacz', 'Magistra', 'vihl', 'Smriti_Iraani', 'boyish_haircut', 'workingwoman', 'borthers', 'Capuchin_friars', 'Nejma', 'yes_sirs', 'bivocational_pastor', 'Grafters', 'HOPWOOD', 'Nicknamed_Godzilla', 'yos', 'Berkenfield', 'Missis', 'sitcom_Designing_Women', 'Kafoa', 'trainer_Emma_Lavelle', 'sadomasochistic_dungeon', 'iht', 'desperates', 'predessor', 'wolf_cub', 'indigenous_Peruvians', 'Livia_Soprano', 'troh', 'colt_sired', 'BOND_HILL', 'ihl', 'Drydens', 'rahs', 'Piserchia', 'Sonny_Corinthos', 'bankrobber', 'Fwank', 'feisty_redhead', 'booze_guzzling', 'COOPERS', "actress_Q'orianka_Kilcher", 'Cortezar', 'twe', 'Jacoub', 'Cindy_Iannarelli', 'Hell_Raiser', 'Fondly_referred', 'Bridal_Shoppe', 'Noleta', 'Christinas', 'IAGRA', 'LaTanya_Richardson', 'Sang_Bender', 'Assasins', 'sorrel_gelding', 'septugenarian', 'Hissy', 'Muqtada_al_Sadr_mook', 'Pfeni', 'MADRID_AFX_Banco_Santander', 'tuchis', 'LeVaughn', 'Gadzicki', 'transvestite_hooker', 'Fame_jockey_Laffit', 'nun_Sister_Mary', 'SAMSONOV', 'Mayflower_Madam', 'Shaque', 'well.He', 'Trainer_Julio_Canani', 'sorrel_mare', 'minivehicle_joint_venture', 'wife_Dwina', "Aasiya_AH'_see", 'Baratheon', "Rick_O'Shay", 'Mammies', 'goatie', 'Nell_Gwynne', 'charmingly_awkward', 'Slamma', 'DEHL', 'Lorenzo_Borghese', 'ALMA_Wis.', 'Anne_Scurria', 'father_Peruvians_alternately', 'JULIE_ANDREWS', 'Slim_Pickins', 'Victoria_Secret_stunner', "BY'", 'Sanam_Devdas', 'pronounced_luh', 'Pasha_Selim', '中华', 'rson', 'maternal_grandmothers', 'IOWA_CITY_Ia', 'Madame_de_Tourvel', "JAY'", 'Sheika_Mozah_bint_Nasser', 'Hotsy_Totsy', "D'_Ginto", 'singer_Johnny_Paycheck', 'uterine_prolapse_surgery', 'SCOTTDALE_Pa.', 'AdelaideNow_reports', 'Marcus_Schenkenberg', 'Clyse', 'Obiter_Dicta', 'comic_Sam_Kinison', 'bitties', 'ROCKVILLE_Ind.', 'swimsuit_calendars', 'Decicio_Smith', 'Ma_ma', 'Rie_Miyazawa', 'celibate_chastity', 'gwah', "ZAY'", 'HER_Majesty', 'Defrere', 'Las_Madrinas', '簿_聂_翻', 'Bea_Hamill', 'ARCADIA_Calif._Trainer', 'Bold_Badgett', 'stakes_victress', 'Hoppin_Frog', 'Narumiya', 'Flayfil', 'hardman_Vinnie_Jones', 'Marilyn_Monroe_lookalike', 'Kivanc_Tatlitug', 'Persis_Khambatta', 'SINKING_SPRING_Pa.', 'len_3rd', 'DEAR_TRYING', 'Farndon_Cheshire', 'Krishna_Madiga', 'daughter_Princess_Chulabhorn', 'Marshall_Rooster_Cogburn', 'Kitty_Kiernan', 'Yokich', 'Jarou', 'Serdaris', 'ee_ay', 'Montifiore', 'Chuderewicz', 'Samuel_Le_Bihan', 'filly_Proud_Spell', 'Umm_Hiba', 'pronounced_koo', 'Sandy_Fonzo', "KOR'", 'Fielder_Civil_kisses', 'Federalsburg_Maryland', 'Nikah_ceremony', 'Brinke_Stevens', 'Yakama_Tribal_Council', 'Capuchin_Father', 'wife_Callista_Bisek', 'Beau_Dare', 'Bedoni', 'Arjun_Punj', 'JOHNNY_KNOXVILLE', 'cap_tain', 'Alderwood_Boys', 'Chi_Eta_Phi', 'ringleader_Charles_Graner', 'Savoies', 'Lalla_Salma', 'Mrs._Potiphar', 'fahn', 'name_Taylor_Sumers', 'Vernita_Green', 'Bollywood_baddie', 'BENBROOK_Texas', 'Assemblyman_Lou_Papan', 'virgin_brides', 'Cho_Eun', 'CATHY_Freeman', 'Uncle_Saul', 'Lao_Brewery', 'Ibo_tribe', 'ruf', 'rival_Edurne_Pasaban', 'Hei_Shangri_La', 'Mommy_dearest', 'interest_Angola_Sonogal', 'Ger_Monsun', 'PUSSYCAT_DOLL', 'Crown_Jewels_Condoms', 'Lord_Marke', 'Patootie', 'Nora_Bey', 'huntin_shootin', 'Minister_Raymond_Tshibanda', 'La_Nina_la_NEEN', 'signature_Whoppers', 'estranged_hubby_Kevin_Federline', "UR'", 'pill_poppin', "GEHR'", 'purebred_Arabians', 'husbandly_duties', 'VIAGRA_TIMING', 'Hereford_heifer', 'hushed_monotone_voice', 'Pola_Uddin', 'Wee_Jimmy_Krankie', 'Kwakwanso', 'Our_Galvinator', 'shoh', 'Codependency_Anonymous_Group', "LA'", "Taufa'ahau", 'Invincible_Spirit_colt', "SAH'_dur", 'MOUNT_CARMEL_Pa.', 'watches_attentively', 'SNL_spinoffs', 'Seth_Nitschke', 'Duns_Berwickshire', 'defendant_Colleen_LaRose', "Silky_O'Sullivan", 'Highcliff_Farm', "REN'", 'Comestar', 'Satisfied_Frog', 'Jai_Maharashtra', 'ATTICA_Ind.', 'lover_Larry_Birkhead', 'Tami_Megal', 'chauvinist_pigs', 'Phi_sorority', 'Micronesian_immigrant', 'Lia_Boldt', 'Sugar_Tits', 'actress_Kathy_Najimy', 'zhoo', 'Colombo_underboss', 'Katsav_accusers', 'Bess_Houdini', 'rap_mogul_Diddy', 'companions_Khin_Khin', 'Van_Het', 'Mastoi_tribe', 'VITALY', 'ROLLING_STONES_rocker', 'womanizing_cad', 'LILY_COLE', 'paternal_grandfathers', 'Lt._Col._Kurt_Kosmatka', 'Kasseem_Jr.', 'Ji_Ji', 'Wilburforce', 'VIAGRA_DOSE', 'English_Sheepdogs', 'pronounced_Kah', 'Htet_Htet_Oo', 'Brisk_Breeze', 'Eau_du', 'BY_MELANIE_EVANS', 'Neovasc_Medical', 'British_funnyman_RICKY', '4YO_mare', 'Hemaida', 'MONKTON', 'Mrs_Mujuru', 'BaGhana_BaGhana', 'Shaaban_Abdel_Rahim', 'Edward_Jazlowiecki_lawyer', 'Ajman_Stud', 'manly_pharaoh_even', 'Serra_Madeira_Islands', "FRAY'", 'panto_dames', 'Khin_Myo', 'dancer_Karima_El_Mahroug', 'CROWN_Princess', 'Baseball_HOFer', 'Hasta_la_Pasta', 'GIRLS_NEXT_DOOR', 'Benedict_Groeschel', 'Bousamra', 'Ruby_Rubacuori_Ruby', 'Monde_Bleu', 'Un_homme_qui', 'Taylor_Sumers', 'Rapper_EMINEM', 'Joe_Menchetti', "VAY'", 'supermodel_NAOMI_CAMPBELL', 'Supermodel_GISELE_BUNDCHEN', 'Au_Lait', 'Radar_Installed', 'THOMAS_TOWNSHIP_Mich.', 'Rafinesque', 'Herman_Weinrich', 'Abraxas_Antelope', 'raspy_voiced_rocker', 'Manurewa_Cosmopolitan_Club', 'Paraone', 'THE_LEOPARD', 'Boy_Incorporated_LZB', 'Dansili_filly', 'Lumpy_Rutherford', 'unwedded_bliss', 'Bhavna_Sharma', 'Scarvagh', 'en_flagrante', 'Mottu_Maid', 'Dowager_Queen', 'NEEN', 'model_Monika_Zsibrita', 'ROSIE_PEREZ', 'Mattock_Ranger', 'Valorous', 'Surpreme', 'Marwari_businessmen', 'Grandparents_aunts', 'Kimberley_Vlaeminck', 'Lyn_Treece_Boys', 'PDX_Update', 'Virsa_Punjab', 'eyelash_fluttering', 'Pi_fraternity', 'HUNTLEIGH_Mo.', 'novelist_Jilly_Cooper', 'Naha_Shuri_temple', 'Yasmine_Al_Massri', 'Mu_Gamma_Xi', 'Mica_Ertegun', 'Ocleppo', 'VIAGRA_CONTRAINDICATIONS', 'daughter_PEACHES', 'trainer_Geoff_Wragg', 'OVERNIGHT_DELIVERY', 'Fitts_retiree', 'de_Tourvel', 'Lil_Lad', 'north_easterner', 'Aol_Weird_News', 'Somewhat_improbably', 'Sikh_panth', 'Worcester_2m_7f', 'Zainab_Jah', 'OLYMPIC_medalist', 'Enoch_Petrucelly', 'collie_Lassie', "LOW'", 'clumsiness_Holloway', 'ayr', "OHR'", 'ROLLING_STONES_guitarist', "LAH'_nee", 'Ian_Beefy_Botham', 'Awapuni_trainer', 'Glamorous_Granny', 'Chiang_Ching', 'MidAtlantic_Cardiovascular_Associates', 'Yeke', 'Seaforth_Huron_Expositor', 'Westley_Cary_Elwes', 'Cate_Blanchett_Veronica_Guerin', 'Bellas_Gate', 'witch_Glinda', 'wives_mistresses', 'Woodsville_Walmart', '2YO_colt', 'Manav_Sushant_Singh', 'Pupi_Avati_Il', 'Sigma_Beta_Rho', 'Bishop_Christopher_Senyonjo', 'Vodou_priest', 'Rubel_Chowdhury', 'Claddagh_Ring', "TAH'_duh_al", "al_Sadr_mook_TAH'", 'ROBIN_GIBB', "GAHN'", 'BY_THOMAS_RANSON', 'sister_Carine_Jena', 'Lyphard_mare', 'summa_cum', 'Semenya_grandmother_Maputhi', 'Clare_Nuns', 'Talac', 'sex_hormones_androgens', 'majeste', 'Saint_Ballado_mare', 'Carrie_Huchel', 'Mae_Dok', 'wife_Dieula', 'Earnest_Sirls', 'spoof_bar_mitzvah', 'von_Boetticher', 'Audwin_Mosby', 'Case_presentationWe', 'Vincent_Papandrea', "KRAY'", 'Sergi_Benavent', 'Le_Poisson', 'Von_Cramm', 'Patti_Mell', 'Raymi_Coya', 'Benjamin_BeBe_Winans', 'Nana_Akosua', 'Auld_Acquaintance', 'Desire_Burunga', 'Company_Wrangler_Nestea', 'ask_Krisy_Plourde', 'JUANITA_BYNUM', 'livia', 'GAMB', 'Gail_Rosario_Dawson', 'Ramgarhia_Sikh', 'Catholic_nun_Sister', 'FOUR_WEDDINGS_AND', 'Robyn_Scherer', 'brother_King_Athelstan', 'Santo_Loquasto_Fences', 'Wee_Frees', 'MARISOL', 'Soliloquy_Stakes', 'Whatever_Spoetzl', "Marc'Aurelio", 'mon_petit', 'Sabbar_al_Mashhadani', "KAY'_lee", "m_zah_MAH'", 'BY_TAMI_ALTHOFF', 'hobbit_Samwise_Gamgee', 'Bahiya_Hariri_sister', 'daddy_Larry_Birkhead', 'Sow_Tracey_Ullman', 'coach_Viljo_Nousiainen', 'Carmen_Lebbos', 'conjoined_twins_Zainab', 'Rob_Komosa', 'ample_bosomed', 'Ageing_rocker', 'psychic_Oda']
    ---------------------------------------------------------------------- 
    


Instantiate and fit the parameters of the debias transformation.
In the fit stage, parameters such as bias direction are calculated and embeddings are 
prepared for the equalization stage.

.. code:: ipython3

    hd = HardDebias(verbose=False, criterion_name="gender")
    
    hd.fit(
        model, definitional_pairs=definitional_pairs, equalize_pairs=equalize_pairs,
    )




.. parsed-literal::

    <wefe.debias.hard_debias.HardDebias at 0x7f3b810ad6d0>



Mitigation Parameters
~~~~~~~~~~~~~~~~~~~~~

The parameters of the transform method are relatively standard for all
methods. The most important ones are ``target``, ``ignore`` and
``copy``.

In the following example we use ``ignore`` and ``copy``, which are
described below:

-  ``ignore`` (by default, ``None``):

    A list of strings that indicates that the debias method will perform
    the debias in all words except those specified in this list. In case
    it is not specified, debias will be executed on all words. In case
    ignore is not specified or its value is None, the transformation will
    be performed on all embeddings. This may cause words that are
    specific to social groups to lose that component (for example,
    leaving ``'she'`` and ``'he'`` without a gender component).

-  ``copy`` (by default ``True``):

    if the value of copy is ``True``, method attempts to create a copy of
    the model and run debias on the copy. If ``False``, the method is
    applied on the original model, causing the vectors to mutate.


.. warning::

    WARNING:** Setting copy with ``True`` requires at least 2x RAM of
    the size of the model. Otherwise the execution of the debias may raise
    ``MemoryError``.

The following transformation is executed using a copy of the model,
ignoring the words contained in ``gender_specific``.


.. code:: ipython3

    gender_debiased_model = hd.transform(model, ignore=gender_specific, copy=True)


.. parsed-literal::

    Copy argument is True. Transform will attempt to create a copy of the original model. This may fail due to lack of memory.
    Model copy created successfully.


.. parsed-literal::

    100%|██████████| 13013/13013 [00:00<00:00, 118668.18it/s]


Measuring the Decrease of Bias
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using the metrics and queries shown in the :ref:`bias measurement` user guide, we
can measure whether there was a change in the measured gender bias
between the original model and the debiased model.

.. code:: ipython3

    from wefe.datasets import load_weat
    from wefe.query import Query
    from wefe.metrics import WEAT
    
    weat_wordset = load_weat()
    weat = WEAT()


Next, we measure the gender bias exposed by query 1 (Male terms and Female terms wrt Career and Family) with respect to the debiased model and the original.

.. code:: ipython3

    gender_query_1 = Query(
        [weat_wordset["male_terms"], weat_wordset["female_terms"]],
        [weat_wordset["career"], weat_wordset["family"]],
        ["Male terms", "Female terms"],
        ["Career", "Family"],
    )
    print(gender_query_1, "\n", "-" * 70, "\n")
    
    biased_results_1 = weat.run_query(gender_query_1, model, normalize=True)
    debiased_results_1 = weat.run_query(
        gender_query_1, gender_debiased_model, normalize=True
    )
    
    print("Debiased vs Biased (absolute values)")
    print(
        round(abs(debiased_results_1["weat"]), 3),
        "<",
        round(abs(biased_results_1["weat"]), 3),
    )
    



.. parsed-literal::

    <Query: Male terms and Female terms wrt Career and Family
    - Target sets: [['male', 'man', 'boy', 'brother', 'he', 'him', 'his', 'son'], ['female', 'woman', 'girl', 'sister', 'she', 'her', 'hers', 'daughter']]
    - Attribute sets:[['executive', 'management', 'professional', 'corporation', 'salary', 'office', 'business', 'career'], ['home', 'parents', 'children', 'family', 'cousins', 'marriage', 'wedding', 'relatives']]> 
     ---------------------------------------------------------------------- 
    
    Debiased vs Biased (absolute values)
    0.047 < 0.463


The above results show that there was a decrease in the measured gender bias.

Next, we measure the gender bias exposed by query 2 (Male Names and Female Names wrt Pleasant and Unpleasant terms) with respect to the debiased model and the original.

.. code:: ipython3

    gender_query_2 = Query(
        [weat_wordset["male_names"], weat_wordset["female_names"]],
        [weat_wordset["pleasant_5"], weat_wordset["unpleasant_5"]],
        ["Male Names", "Female Names"],
        ["Pleasant", "Unpleasant"],
    )
    
    print(gender_query_2, "\n", "-" * 70, "\n")
    
    biased_results_2 = weat.run_query(
        gender_query_2, model, normalize=True, preprocessors=[{}, {"lowercase": True}]
    )
    debiased_results_2 = weat.run_query(
        gender_query_2,
        gender_debiased_model,
        normalize=True,
        preprocessors=[{}, {"lowercase": True}],
    )
    
    print("Debiased vs Biased (absolute values)")
    print(
        round(abs(debiased_results_2["weat"]), 3),
        "<",
        round(abs(biased_results_2["weat"]), 3),
    )
    



.. parsed-literal::

    <Query: Male Names and Female Names wrt Pleasant and Unpleasant
    - Target sets: [['John', 'Paul', 'Mike', 'Kevin', 'Steve', 'Greg', 'Jeff', 'Bill'], ['Amy', 'Joan', 'Lisa', 'Sarah', 'Diana', 'Kate', 'Ann', 'Donna']]
    - Attribute sets:[['caress', 'freedom', 'health', 'love', 'peace', 'cheer', 'friend', 'heaven', 'loyal', 'pleasure', 'diamond', 'gentle', 'honest', 'lucky', 'rainbow', 'diploma', 'gift', 'honor', 'miracle', 'sunrise', 'family', 'happy', 'laughter', 'paradise', 'vacation'], ['abuse', 'crash', 'filth', 'murder', 'sickness', 'accident', 'death', 'grief', 'poison', 'stink', 'assault', 'disaster', 'hatred', 'pollute', 'tragedy', 'divorce', 'jail', 'poverty', 'ugly', 'cancer', 'kill', 'rotten', 'vomit', 'agony', 'prison']]> 
     ---------------------------------------------------------------------- 
    
    Debiased vs Biased (absolute values)
    0.055 < 0.074


Again, the above results show that there was a decrease in the measured gender bias.

Target Parameter
~~~~~~~~~~~~~~~~

If a set of words is specified in ``target`` parameter, the debias method is performed
only on the embeddings associated with this set. 
In the case of providing ``None``, the transformation is performed on all vocabulary
words except those specified in ignore. By default ``None``.

In the following example, the target parameter is used to execute the transformation 
only on the career and family word set:

.. code:: ipython3

    targets = [
        "executive",
        "management",
        "professional",
        "corporation",
        "salary",
        "office",
        "business",
        "career",
        "home",
        "parents",
        "children",
        "family",
        "cousins",
        "marriage",
        "wedding",
        "relatives",
    ]
    
    hd = HardDebias(verbose=False, criterion_name="gender").fit(
        model, definitional_pairs=definitional_pairs, equalize_pairs=equalize_pairs,
    )
    
    gender_debiased_model = hd.transform(model, target=targets, copy=True)



.. parsed-literal::

    Copy argument is True. Transform will attempt to create a copy of the original model. This may fail due to lack of memory.
    Model copy created successfully.


.. parsed-literal::

    100%|██████████| 16/16 [00:00<00:00, 9428.05it/s]


Next, a bias test is run on the mitigated embeddings associated with the
target words. 

In this case, the value of the metric is lower on the
query executed on the mitigated model than on the original one.
These results indicate that there was a mitigation of bias on embeddings of these words.


.. code:: ipython3

    gender_query_1 = Query(
        [weat_wordset["male_terms"], weat_wordset["female_terms"]],
        [weat_wordset["career"], weat_wordset["family"]],
        ["Male terms", "Female terms"],
        ["Career", "Family"],
    )
    print(gender_query_1, "\n", "-" * 70, "\n")
    
    biased_results_1 = weat.run_query(gender_query_1, model, normalize=True)
    debiased_results_1 = weat.run_query(
        gender_query_1, gender_debiased_model, normalize=True
    )
    
    print("Debiased vs Biased (absolute values)")
    print(
        round(abs(debiased_results_1["weat"]), 3),
        "<",
        round(abs(biased_results_1["weat"]), 3),
    )
    



.. parsed-literal::

    <Query: Male terms and Female terms wrt Career and Family
    - Target sets: [['male', 'man', 'boy', 'brother', 'he', 'him', 'his', 'son'], ['female', 'woman', 'girl', 'sister', 'she', 'her', 'hers', 'daughter']]
    - Attribute sets:[['executive', 'management', 'professional', 'corporation', 'salary', 'office', 'business', 'career'], ['home', 'parents', 'children', 'family', 'cousins', 'marriage', 'wedding', 'relatives']]> 
     ---------------------------------------------------------------------- 
    
    Debiased vs Biased (absolute values)
    0.047 < 0.463


However, if a bias test is run with words that were outside the ``target``
word set, the results are almost the same. The slight difference in the
metric scores lies in the fact that the equalize sets were still
equalized.

.. warning::

    The equalization process can modify embeddings that have not been marked in the target.
    In Hard Debias, equalization can be deactivated by delivering an empty 
    equalize set (``[]``).



.. code:: ipython3

    gender_query_2 = Query(
        [weat_wordset["male_names"], weat_wordset["female_names"]],
        [weat_wordset["pleasant_5"], weat_wordset["unpleasant_5"]],
        ["Male Names", "Female Names"],
        ["Pleasant", "Unpleasant"],
    )
    
    print(gender_query_2, "\n", "-" * 70, "\n")
    
    biased_results_2 = weat.run_query(
        gender_query_2, model, normalize=True, preprocessors=[{}, {"lowercase": True}]
    )
    debiased_results_2 = weat.run_query(
        gender_query_2,
        gender_debiased_model,
        normalize=True,
        preprocessors=[{}, {"lowercase": True}],
    )
    
    print("Debiased vs Biased (absolute values)")
    print(
        round(abs(debiased_results_2["weat"]), 3),
        ">",
        round(abs(biased_results_2["weat"]), 3),
    )



.. parsed-literal::

    <Query: Male Names and Female Names wrt Pleasant and Unpleasant
    - Target sets: [['John', 'Paul', 'Mike', 'Kevin', 'Steve', 'Greg', 'Jeff', 'Bill'], ['Amy', 'Joan', 'Lisa', 'Sarah', 'Diana', 'Kate', 'Ann', 'Donna']]
    - Attribute sets:[['caress', 'freedom', 'health', 'love', 'peace', 'cheer', 'friend', 'heaven', 'loyal', 'pleasure', 'diamond', 'gentle', 'honest', 'lucky', 'rainbow', 'diploma', 'gift', 'honor', 'miracle', 'sunrise', 'family', 'happy', 'laughter', 'paradise', 'vacation'], ['abuse', 'crash', 'filth', 'murder', 'sickness', 'accident', 'death', 'grief', 'poison', 'stink', 'assault', 'disaster', 'hatred', 'pollute', 'tragedy', 'divorce', 'jail', 'poverty', 'ugly', 'cancer', 'kill', 'rotten', 'vomit', 'agony', 'prison']]> 
     ---------------------------------------------------------------------- 
    
    Debiased vs Biased (absolute values)
    0.08 > 0.074


Note that the equalization caused the bias of the debiased model to be slightly larger than the original.


Saving the Debiased Model
~~~~~~~~~~~~~~~~~~~~~~~~~

To save the mitigated model one must access the ``KeyedVectors`` (the
gensim object that contains the embeddings) through ``wv`` and then use
the ``save`` method to store the method in a file.



.. code:: ipython3

    gender_debiased_model.wv.save("gender_debiased_glove.kv")
    


Multiclass Hard Debias
----------------------

Multiclass Hard Debias is a generalized version of Hard Debias that
enables multiclass debiasing. Generalized refers to the fact that this
method extends Hard Debias in order to support more than two types of
social target sets within the definitional set.

For example, for the case of religion bias, it supports a debias using
words associated with Christianity, Islam and Judaism.

The usage is very similar to Hard Debias with the difference that the
``definitional_sets`` can be larger than pairs.


.. code:: ipython3

    from wefe.datasets import fetch_debias_multiclass
    from wefe.debias.multiclass_hard_debias import MulticlassHardDebias
    
    multiclass_debias_wordsets = fetch_debias_multiclass()
    weat_wordsets = load_weat()
    weat = WEAT()
    
    ethnicity_definitional_sets = multiclass_debias_wordsets["ethnicity_definitional_sets"]
    ethnicity_equalize_sets = list(
        multiclass_debias_wordsets["ethnicity_analogy_templates"].values()
    )
    
    print(f"ethnicity_definitional_sets: \n{ethnicity_definitional_sets}")
    print(f"ethnicity_equalize_sets: \n{ethnicity_equalize_sets}")
    print("-" * 70, "\n")
    
    mhd = MulticlassHardDebias(verbose=False, criterion_name="ethnicity")
    mhd.fit(
        model=model,
        definitional_sets=ethnicity_definitional_sets,
        equalize_sets=ethnicity_equalize_sets,
    )
    
    ethnicity_debiased_model = mhd.transform(model, copy=True)



.. parsed-literal::

    ethnicity_definitional_sets: 
    [['black', 'caucasian', 'asian'], ['african', 'caucasian', 'asian'], ['black', 'white', 'asian'], ['africa', 'america', 'asia'], ['africa', 'america', 'china'], ['africa', 'europe', 'asia']]
    ethnicity_equalize_sets: 
    [['manager', 'executive', 'redneck', 'hillbilly', 'leader', 'farmer'], ['doctor', 'engineer', 'laborer', 'teacher'], ['slave', 'musician', 'runner', 'criminal', 'homeless']]
    ---------------------------------------------------------------------- 
    
    copy argument is True. Transform will attempt to create a copy of the original model. This may fail due to lack of memory.
    Model copy created successfully.


.. parsed-literal::

    100%|██████████| 13003/13003 [00:00<00:00, 18357.20it/s]


Measuring the Decrease of Bias
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following code compares the execution of a query measuring ethnic bias in the original model vs. in the debiased model.

.. code:: ipython3

    ethnicity_query = Query(
        [
            multiclass_debias_wordsets["white_terms"],
            multiclass_debias_wordsets["black_terms"],
        ],
        [
            multiclass_debias_wordsets["white_biased_words"],
            multiclass_debias_wordsets["black_biased_words"],
        ],
        ["european_american_names", "african_american_names"],
        ["white_biased_words", "black_biased_words"],
    )
    
    print(ethnicity_query, "\n", "-" * 70, "\n")
    
    biased_results = weat.run_query(
        ethnicity_query, model, normalize=True, preprocessors=[{}, {"lowercase": True}],
    )
    debiased_results = weat.run_query(
        ethnicity_query,
        ethnicity_debiased_model,
        normalize=True,
        preprocessors=[{}, {"lowercase": True}],
    )
    
    print("Debiased vs Biased (absolute values)")
    print(
        round(abs(debiased_results_2["weat"]), 3),
        "<",
        round(abs(biased_results_2["weat"]), 3),
    )



.. parsed-literal::

    <Query: european_american_names and african_american_names wrt white_biased_words and black_biased_words
    - Target sets: [['america', 'caucasian', 'europe', 'white'], ['africa', 'african', 'black']]
    - Attribute sets:[['manager', 'executive', 'redneck', 'hillbilly', 'leader', 'farmer'], ['slave', 'musician', 'runner', 'criminal', 'homeless']]> 
     ---------------------------------------------------------------------- 
    
    Debiased vs Biased (absolute values)
    0.08 < 0.074

