## chord-wheel-app

> In music theory, the circle of fifths (sometimes also cycle of fifths) is a way of organizing pitches as a sequence of perfect fifths. Starting on a C, and using the standard system of tuning for Western music (12-tone equal temperament), the sequence is: C, G, D, A, E, B, F♯/G♭, C♯/D♭, G♯/A♭, D♯/E♭, A♯/B♭, F, and C. This order places the most closely related key signatures adjacent to one another.


In music theory, the circle of fifths (sometimes also cycle of fifths) is a way of organizing pitches as a sequence of perfect fifths. Starting on a C, and using the standard system of tuning for Western music (12-tone equal temperament), the sequence is: C, G, D, A, E, B, F♯/G♭, C♯/D♭, G♯/A♭, D♯/E♭, A♯/B♭, F, and C. This order places the most closely related key signatures adjacent to one another.

Twelve-tone equal temperament tuning divides each octave into twelve equivalent semitones, and the circle of fifths leads to a C seven octaves above the starting point. If the fifths are tuned with an exact frequency ratio of 3:2 (the system of tuning known as just intonation), this is not the case (the circle does not "close").

Definition
The circle of fifths organizes pitches in a sequence of perfect fifths, generally shown as a circle with the pitches (and their corresponding keys) in clockwise order. It can be viewed in a counterclockwise direction as a circle of fourths. Harmonic progressions in Western music commonly use adjacent keys in this system, making it a useful reference for musical composition and harmony.[1]

The top of the circle shows the key of C Major, with no sharps or flats. Proceeding clockwise, the pitches ascend by fifths. The key signatures associated with those pitches change accordingly: the key of G has one sharp, the key of D has 2 sharps, and so on. Proceeding counterclockwise from the top of the circle, the notes change by descending fifths and the key signatures change accordingly: the key of F has one flat, the key of B♭ has 2 flats, and so on. Some keys (at the bottom of the circle) can be notated either in sharps or in flats.

Starting at any pitch and ascending by a fifth generates all tones before returning to the beginning pitch class (a pitch class consists of all of the notes indicated by a given letter regardless of octave—all "C"s, for example, belong to the same pitch class). Moving counterclockwise, the pitches descend by a fifth, but ascending by a perfect fourth will lead to the same note an octave higher (therefore in the same pitch class). Moving counter-clockwise from C could be thought of as descending by a fifth to F, or ascending by a fourth to F.

 {
\omit Score.TimeSignature
\relative c' { \set Score.tempoHideNote = ##t \tempo 1 = 60 \time 12/1
  c1 g' d a' e b' fis cis gis' dis ais' f | c! \bar "|."
} }
\layout { \context {\Score \omit BarNumber} line-width = #100 }
Circle of fifths clockwise within one octave
  {
\omit Score.TimeSignature
\relative c' { \set Score.tempoHideNote = ##t \tempo 1 = 60 \time 12/1
  c1 f bes ees, aes des, ges b e, a d, g | c, \bar "|."
} }
\layout { \context {\Score \omit BarNumber} line-width = #100 }
Circle of fifths counterclockwise within one octave

Structure and use
Diatonic key signatures
Each pitch can serve as the tonic of a major or minor key, and each of these keys will have a diatonic scale associated with it. The circle diagram shows the number of sharps or flats in each key signature, with the major key indicated by a capital letter and the minor key indicated by a lower-case letter. Major and minor keys that have the same key signature are referred to as relative major and relative minor of one another.

Modulation and chord progression
Tonal music often modulates to a new tonal center whose key signature differs from the original by only one flat or sharp. These closely-related keys are a fifth apart from each other and are therefore adjacent in the circle of fifths. Chord progressions also often move between chords whose roots are related by perfect fifth, making the circle of fifths useful in illustrating the "harmonic distance" between chords.


Major 7th progressing on umbilic torus surface
The circle of fifths is used to organize and describe the harmonic or tonal function of chords.[2] Chords can progress in a pattern of ascending perfect fourths (alternately viewed as descending perfect fifths) in "functional succession". This can be shown "...by the circle of fifths (in which, therefore, scale degree II is closer to the dominant than scale degree IV)".[3] In this view the tonic or tonal center is considered the end point of a chord progression derived from the circle of fifths.


ii–V–I progression, in C, illustrating the similarity between them
Duration: 0 seconds.0:00
Subdominant, supertonic seventh, and supertonic chords
According to Richard Franko Goldman's Harmony in Western Music, "the IV chord is, in the simplest mechanisms of diatonic relationships, at the greatest distance from I. In terms of the [descending] circle of fifths, it leads away from I, rather than toward it."[4] He states that the progression I–ii–V–I (an authentic cadence) would feel more final or resolved than I–IV–I (a plagal cadence). Goldman[5] concurs with Nattiez, who argues that "the chord on the fourth degree appears long before the chord on II, and the subsequent final I, in the progression I–IV–viio–iii–vi–ii–V–I", and is farther from the tonic there as well.[6] (In this and related articles, upper-case Roman numerals indicate major triads while lower-case Roman numerals indicate minor triads.)

Circle closure in non-equal tuning systems
Using the exact 3:2 ratio of frequencies to define a perfect fifth (just intonation) does not quite result in a return to the pitch class of the starting note after going around the circle of fifths. Twelve-tone equal temperament tuning produces fifths that return to a tone exactly seven octaves above the initial tone and makes the frequency ratio of the chromatic semitone the same as that of the diatonic semitone. The standard tempered fifth has a frequency ratio of 27/12:1 (or about 1.498307077:1), approximately two cents narrower than a justly tuned fifth.

Ascending by twelve justly tuned fifths fails to close the circle by an excess of approximately 23.46 cents, roughly a quarter of a semitone, an interval known as the Pythagorean comma. If limited to twelve pitches per octave, Pythagorean tuning markedly shortens the width of one of the twelve fifths, which makes it severely dissonant. This anomalous fifth is called the wolf fifth – a humorous reference to a wolf howling an off-pitch note. Non-extended quarter-comma meantone uses eleven fifths slightly narrower than the equally tempered fifth, and requires a much wider and even more dissonant wolf fifth to close the circle. More complex tuning systems based on just intonation, such as 5-limit tuning, use at most eight justly tuned fifths and at least three non-just fifths (some slightly narrower, and some slightly wider than the just fifth) to close the circle.

Equal-tempered tunings with more than twelve notes
Nowadays, with the advent of electronic isomorphic keyboards, equal temperament tunings with more than twelve notes per octave can be used to close the circle of fifths for other tunings. For example, 31-tone equal temperament closely approximates quarter-comma meantone, and 53-tone equal temperament closely approximates Pythagorean tuning.


Basic Use

The Circle starts out with the key of C Major selected. To choose a different tonic, click an item in the Tonic table with your mouse. Likewise, to choose a different mode, click in the Mode table. To save a lot of clicking, you can also rotate the Circle clockwise by dragging upwards in either table, and counter-clockwise by dragging downwards.

The white rows in the Tonic table correspond to the fifteen classic Major key signatures that we learn about in music class, from C (seven sharps) through C (seven flats). The gray rows are more rarely used, and are included here for completeness. (When's the last time you heard a tune in E Phrygian?)

Note on Minor Keys: The Mode table contains the entry "N. Minor / Aeolian". This refers to the natural minor scale, which consists of the same notes as the Aeolian mode. Unfortunately, in actual practice, music in the minor mode is rarely pure Aeolian. Instead, it typically uses a major V chord rather than the minor one called for by the Aeolian mode. It will also often include the ascending melodic minor scale. The details are beyond the scope of this Guide, but you should at least be aware of these complications so you're not surprised when you come across them in music.

To make matters worse, you may encounter the term "minor" in a number of different contexts: Harmonic minor, ascending and descending melodic minor, relative minor, and parallel minor. In this Guide, the term "minor mode" always refers to the natural minor in its pure Aeolian form, since this is the only one that fits entirely into the structure of the circle of fifths.


As you can see, the Circle is made up of three concentric rings. The large middle ring, the Note Ring, shows the names of all of the notes in the 12-tone chromatic scale. The seven notes in the key you have selected (that is, diatonic to the key) have a white background, and the other five notes have a gray background.

The innermost ring, the Degree Ring, gives, in Roman numerals, the scale degrees of the seven notes highlighted in the Note Ring. (The degree of a note is just its position in the scale, but this becomes very important when we work with chord progressions.) You'll also see a black arrow pointing out the tonic of the selected key.

The outermost ring, the Chord Ring, shows you what type of three-note chord, or triad, is rooted at each of the seven notes in the selected key. For example, in C Major, the triad rooted at D (that is, D-F-A) is a minor chord, but in G Major, the triad rooted at D (D-F-A) is instead a major chord.


Example 1: Start with the Circle in its starting position, with C Major selected. The Note Ring shows that the notes F, C, G, D, A, E, B are diatonic to C Major. (That is, C Major has no sharps or flats in its key signature.) Put them in alphabetical order, and you have the scale C, D, E, F, G, A, B. The Degree Ring indicates the tonic (C) with a black arrow, and also shows, for instance, that G is the fifth, or dominant of this key (more on degrees later). The Chord Ring shows that in C Major, F, C, and G are major chords, D, A, and E are minor chords, and B is a diminished chord.

Example 2: Now click G in the Tonic table, immediately above C. The circle rotates clockwise one position, and the Degree Ring now points to G as the tonic. Notice that F has dropped off one end of the Note Ring, and F has been highlighted at the other end. This is an important lesson about the Circle: When you rotate it (whether by Tonic or Mode), you are simply taking a note at one end of the Note Ring and sharpening it (clockwise rotation) or flattening it (counterclockwise rotation). The other six notes remain the same. This means that the closer two keys are on the circle, the more notes they have in common. It also tells you, if you're searching for a key with more sharps, go clockwise; if you're looking for more flats, go counterclockwise. (Using the Tonic and Mode tables, remember that flat is down and sharp is up...just as in music.)

Example 3: Now use the Tonic and Mode tables to select C Lydian. Notice that we've gone down one spot on the Tonic table and up one spot on the Mode table, and the Circle looks almost the same as it did in Example 2: The Note and Chord Rings are the same, but the Degree Ring has changed. C is now the tonic, and the other degrees have changed accordingly. From this, we can gather that changing modes is very much like changing from one tonic to another, and in fact, if we offset a change in tonic with an opposite change in mode, the new key will have the same notes as the old key (the keys are enharmonic).

Analyzing a Chord Progression
Analyzing a chord progression has two basic steps: Figuring out what key to use, and then figuring out the degrees of the chords.

Example 1: The chords for the Eagles tune "Take It Easy" are as follows:
G	D	C	G	D	C	G	Em	C	G
Am	C	Em	C	G	C	G	Am	C	G

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/apo-bozdag) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
