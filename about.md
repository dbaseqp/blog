---
layout: landing
titles:
  # @start locale config
  en      : &EN       About Me
  en-GB   : *EN
  en-US   : *EN
  en-CA   : *EN
  en-AU   : *EN
  zh-Hans : &ZH_HANS  关于我
  zh      : *ZH_HANS
  zh-CN   : *ZH_HANS
  zh-SG   : *ZH_HANS
  zh-Hant : &ZH_HANT  關於我
  zh-TW   : *ZH_HANT
  zh-HK   : *ZH_HANT
  ko      : &KO       소개
  ko-KR   : *KO
  fr      : &FR       À propos
  fr-BE   : *FR
  fr-CA   : *FR
  fr-CH   : *FR
  fr-FR   : *FR
  fr-LU   : *FR
  # @end locale config
key: page-about
#full_width: false
header: true

article_header:
  height: 100vh
  theme: dark
  background_color: "#367a9a"
  background_image:
    gradient: "linear-gradient(rgba(0, 0, 0, .2), rgba(0, 0, 0, .6))"
    src: /assets/images/logo/logo.png
data:
  sections:
    - title: Gabriel Fok <i class="fas fa-shield-alt"></i>
      excerpt: >
        Also known as baseq.
        <br>
        I'm a security enthusiest, avid esports fan, and learner of just about anything I find interesting.
        <br>
        I represent my university at security competitions as the CCDC Linux Security lead and a member of the CPTC team. 
        <br>
        I like learning to spread my knowledge and experience.
        <br>
        Current director of SWIFT Academy.
        <br>
        <br>
        Currently a Computer Science Undergraduate at Cal Poly Pomona. <i class="fas fa-horse-head"></i>
      actions:
        - text: Dotfiles
          url: https://github.com/dbaseqp/dotfiles
    - title: My Certifications <i class="fas fa-graduation-cap"></i>
      excerpt: >
        CompTIA A+
        <br>
    - title: My Competition Achievements <i class="fas fa-trophy"></i>
      children:
        - title: Collegiete Cybersecurity Defense Competition
          excerpt: >
            9th Place | WRCCDC Qualifiers | 2020
            <br>
        - title: Collegiete Penetration Testing Competition
          excerpt: >
            1st Place | WRCPTC Regionals  | 2021
            1st Place | CPTC World Finals | 2022
        - title: CyberPatriot
          excerpt: >
            <br>
            4th Place | CyberPatriot Nationals | 2020
            <br>
            6th Place | CyberPatriot Nationals | 2019
---
