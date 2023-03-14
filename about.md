---
layout: article
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
certifications:
  - name: Associate of (ISC)<sup>2</sup>
  - name: CompTIA A+
competitions:
  - award: 4th Place
    name: CCDC Western Qualifiers
    year: 2022-2023
  - award: 1st Place
    name: CPTC Global Finals
    year: 2022-2023
  - award: 2nd Place
    name: CCDC Western Invitational
    year: 2022-2023
  - award: 1st Place
    name: CPTC Western Regionals
    year: 2022-2023
  - award: 7th Place 
    name: Hivestorm 
    year: 2022-2023
  - award: 1st Place
    name: CPTC Global Finals
    year: 2021-2022
  - award: 1st Place
    name: CPTC Western Regionals
    year: 2021-2022
  - award: 8th Place 
    name: CCDC Western Regionals 
    year: 2021-2022
  - award: 7th Place 
    name: CCDC Western Qualifiers 
    year: 2021-2022
  - award: 9th Place 
    name: Hivestorm 
    year: 2020-2021
  - award: 9th Place
    name: CCDC Western Qualifiers
    year: 2020-2021
  - award: 6th Place 
    name: Hivestorm 
    year: 2020-2021
  - award: 4th Place
    name: CyberPatriot Nationals 
    year: 2019-2020
  - award: 6th Place
    name: CyberPatriot Nationals 
    year: 2018-2019
---
<div id=about>
  <h3 ><i>Computer Science Undergraduate Student at Cal Poly Pomona. </i></h3>
  <div>
    <p>
      Some people know me as baseq. I'm a person with many interests&#8212;security, esports, and music to name a few.
    </p>

    <p>
      Code monkey and *nix nerd for various security competitions I've participated in. Currently captain of the CCDC team and former co-captain of the CPTC
      team. I serve as the vice president of operations at
      <a href="https://www.calpolyswift.org/" target="_blank">Students With an Interest in the Future of Technology</a>
      where I try to spread my knowledge and experience to the community.
    </p>
  </div>

  <div>
    <div>
      <h2>Certifications</h2>
    </div>
    <div class="certifications">
    <ul>
      {%- for _certification in page.certifications -%}
        <li>
        <div class="cert">
<p>{{_certification.name }}</p>
          {%- if _certification.src -%}
<img class="image image--md" src="{{_certification.src }}">
          {%- endif -%}
        </div>
        </li>
      {%- endfor -%}
    </ul>
  </div>
  
  <div>
    <div>
      <h2>Competitive Achievements</h2>
    </div>
    <table class="competitions">
      <thead>
          <tr>
              <th>Placement</th>
              <th>Competition</th>
              <th>Season</th>
          </tr>
      </thead>
      <tbody>
        {%- for _competition in page.competitions -%}
          <tr>
            <td><b>{{_competition.award }}</b></td>
            <td>{{ _competition.name }}</td>
            <td>{{_competition.year}}</td>
          </tr>
        {%- endfor -%}
      </tbody>
    </table>
  </div>
</div>
