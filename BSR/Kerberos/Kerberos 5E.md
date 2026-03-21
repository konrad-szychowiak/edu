![[BSR/Kerberos/README#^Kerberos-intor]]

## Engage

> [!TIP]- Engage
> Engage is the first phase of the 5E model. During this phase, teachers are activating students’ prior knowledge to identify what students know or do not know about the upcoming concept. As teachers tap into students’ background knowledge, students can make connections and teachers can identify any knowledge gaps. This phase also serves to pique students’ interest and curiosity about the topic at hand. To engage students, ask open-ended questions, lead a class discussion, or view videos to introduce a concept.

![[Pasted image 20260316214122.png]]

> [!TODO]
> Jako engage pytania do Krb i do sensu jego działania.

## Explore

> [!TIP]- Explore
> During the Explore phase, teachers are guiding students in exploration and problem-solving in a concrete way. Through hands-on activities, such as creating models or conducting experiments, students can investigate the new concept and discuss ideas and observations with their peers.

> [!TODO] Playground
> Docker compose, który uruchomi wszystkie trzy role na raz. Studenci będą mogli sprawdzić jak działa ich system i się nim trochę pobawić. 
> Kontenery mogą być gotowe z https://github.com/criteo/kerberos-docker/tree/main albo postawione od podstaw na obrazie [OpenSUSE](https://hub.docker.com/r/opensuse/leap). 
> Fajnie jakby obraz/kontener pozwalał łatwo załadować nową konfigurację *a może nawet ją przetestować?*.

### Kontener KDC
![[BSR/Kerberos/KDC|KDC]]

### Kontener Client
![[BSR/Kerberos/Client|Client]]

### Kontener Serwera Usługi
![[BSR/Kerberos/Server|Server]]

## Explain

> [!TIP]- Explain
> The Explain phase is run by the teacher. During this phase, the teacher facilitates a whole-class discussion by asking questions, comparing student responses, and helping to guide the class towards the key ideas being taught.

> [!TODO]
> Miejsce na rozwinięcie informacji o tym czym w ogóle jest Krb, KDC itp. Może w miarę follow along takiego, że część wyjaśnienia prowadzi od razu do "spójrzmy teraz jak to jest zrobione w naszej aplikacji..."

## Elaborate

> [!TIP]- Elaborate
> During the Elaborate phase, students have the space to apply what they learned. They can take their new knowledge to form a new hypothesis, explore real-world scenarios, or create a presentation to share with their peers. This phase allows students to extend their learning and create richer connections to concepts.

Elaborate w tym przypadku składa się z 3 "klocków", z których każdy może być 'reversible':
- wymiana serwera usług na inny
- wymiana klienta który się uwierzytelnia z kontenera na lokalny system
- wymiana KDC na taki na hoście lokalnym lub zdalnym komputerze (partnera)

## Evaluate

> [!TIP]- Evaluate
> At this phase, the teacher assesses student learning through formal and/or informal assessments. Informal assessments, like exit tickets or oral presentations, or formal assessments, like tests or quizzes, can be used to determine whether students understood the key concepts. During this phase, students can also evaluate their learning using self-assessment tools like rubrics.

Tutaj chciałbym im dać ankietę, żeby ocenili to rozwiązanie i z czym mieli problemy a co było sensowne. Chciałbym też ocenić na ile udało im się ogólnie dojechać do końca.
