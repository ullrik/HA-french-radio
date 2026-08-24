# HA-French-radio
## Descritpion
Voici le code nécessaire pour mettre des stations de radio (ex : radio Française) sur votre media player (ex : google_home).

## Note
Je ne suis pas le création des scripts initiaux, je l'avais dans ma configuration depuis plusieurs années. Par contre, j'ai corrigé les différents bugs que j'avais dans cette solutions que je partage.

## Home Assistant
### Fichiers
Dans le répertoire www de Home Assistant mettre le contenu de www dans les sources Github
### Configuration
Dans Configuration.yaml ou dans votre input_select.yaml (si vous avez externaliser la conf) :
```
input_select:
  radio_station_google_home:
    name: 'Radio google home'
    options:
      - Aucune
      - Skyrock
      - NRJ
      - Fun
      - Nostalgie
      - BFM
      - Virgin
      - FG
      - MTI
      - Scoop
      - Alouette
      - HitWest
      - VirginRadio
      - Hitsradio
    icon: mdi:radio
```

### Script
Diminuer le volume
```
alias: '[Google Home][Radio] Down volume on Media Player'
sequence:
  - data_template:
      entity_id: |
        media_player.{{ media }}
      volume_level: >
        {% set next_vol = states['media_player.' +
        media].attributes.volume_level | float - 0.02 %}
          {{ next_vol }}
          
    action: media_player.volume_set
description: ''
```

Augmenter le volume
```
alias: '[Google Home][Radio] Up volume on Media Player'
sequence:
  - data_template:
      entity_id: |
        media_player.{{ media }}
      volume_level: >
        {% set next_vol = states['media_player.' +
        media].attributes.volume_level | float + 0.02 %}
          {{ next_vol }}
          
    action: media_player.volume_set
description: ''
```

Lecture d'une radio (dans la liste construite ci-dessous)
```
alias: '[Google Home][Radio] Play Radio on Media Player'
sequence:
  - data_template:
      entity_id: |
        input_select.radio_station_google_home
      option: '{{ radio }}'
    action: input_select.select_option
  - data_template:
      entity_id: |
        media_player.{{ media }}
      media_content_id: >
        {% if(radio == "Skyrock") %} http://icecast.skyrock.net/s/natio_mp3_128k
        {% elif(radio == "NRJ") %}
        http://cdn.nrjaudio.fm/audio1/fr/30001/mp3_128.mp3?origine=fluxradios {%
        elif(radio == "Fun") %}
        http://icecast.funradio.fr/fun-1-44-128?listen=webCwsBCggNCQgLDQUGBAcGBg
        {% elif(radio == "Nostalgie") %}
        https://streaming.nrjaudio.fm/oug7girb92oc?origine=fluxurlradio {%
        elif(radio == "BFM") %} http://audio.bfmtv.com/bfmbusiness_128.mp3 {%
        elif(radio == "MRadio") %} http://mfm.ice.infomaniak.ch/mfm-128.mp3 {%
        elif(radio == "FG") %} http://radiofg.impek.com/fg.mp3 {% elif(radio ==
        "MTI") %}    http://radiomti.ice.infomaniak.ch/radiomti.mp3 {%
        elif(radio == "Scoop") %}
        http://radioscooplyon.ice.infomaniak.ch/radioscoop-lyon-128.mp3 {%
        elif(radio == "VirginRadio") %}
        http://virginradio.ice.infomaniak.ch/virgin-radio.mp3 {% elif(radio ==
        "HitWest") %} https://stream.rcs.revma.com/9nbpzyu59k3vv {% elif(radio
        == "Alouette") %} http://alouette.ice.infomaniak.ch/alouette-high.mp3 {%
        elif(radio == "Hitsradio") %}
        http://streamlive.syndicationradio.fr:8072/; {% endif %}
      media_content_type: audio/mp3
    action: media_player.play_media
  - data_template:
      entity_id: |
        media_player.{{ media }}
    action: media_player.media_pause
  - delay:
      milliseconds: 800
  - data_template:
      entity_id: media_player.{{ media }}
    action: media_player.media_play
description: ''
```

Arrêt de la lecture
```
alias: '[Google Home][Radio] Stop Radio on Media Player'
sequence:
  - data_template:
      entity_id: |
        media_player.{{ media }}
    action: media_player.media_stop
  - data_template:
      entity_id: |
        input_select.radio_station_google_home
      option: Aucune
    action: input_select.select_option
description: ''
```

### Lovelace
Dans un bord HA : 
```
type: vertical-stack
cards:
  - cards: null
    type: conditional
    conditions:
      - entity: media_player.station_google_home
        state_not: 'off'
      - entity: media_player.station_google_home
        state_not: unavailable
    card:
      artwork: cover
      entity: media_player.station_google_home
      hide:
        icon_state: false
        name: true
        info: true
        power: true
        power_state: false
        play_pause: true
      icon: mdi:google-home
      max_volume: '100'
      min_volume: '1'
      shortcuts:
        buttons:
          - data:
              media: station_google_home
            icon: mdi:volume-minus
            id: script.down_vol_radio
            type: service
          - data:
              media: station_google_home
            icon: mdi:volume-plus
            id: script.up_vol_radio
            type: service
        columns: 2
        hide_when_off: true
      type: custom:mini-media-player
      volume_stateless: false
      toggle_power: true
      group: false
  - cards: null
    type: conditional
    conditions:
      - entity: media_player.station_google_home
        state_not: 'off'
      - entity: media_player.station_google_home
        state_not: unavailable
    card:
      cards:
        - card:
            image: /local/img/radio/Alouette_selected.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.stop_radio
              service_data:
                media: station_google_home
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state: Alouette
          type: conditional
        - card:
            image: /local/img/radio/Alouette.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.start_radio
              service_data:
                media: station_google_home
                radio: Alouette
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state_not: Alouette
          type: conditional
        - card:
            image: /local/img/radio/HitWest_selected.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.stop_radio
              service_data:
                media: station_google_home
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state: HitWest
          type: conditional
        - card:
            image: /local/img/radio/HitWest.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.start_radio
              service_data:
                media: station_google_home
                radio: HitWest
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state_not: HitWest
          type: conditional
        - card:
            image: /local/img/radio/VirginRadio_selected.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.stop_radio
              service_data:
                media: station_google_home
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state: VirginRadio
          type: conditional
        - card:
            image: /local/img/radio/VirginRadio.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.start_radio
              service_data:
                media: station_google_home
                radio: VirginRadio
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state_not: VirginRadio
          type: conditional
        - card:
            image: /local/img/radio/NRJ_selected.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.stop_radio
              service_data:
                media: station_google_home
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state: NRJ
          type: conditional
        - card:
            image: /local/img/radio/NRJ.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.start_radio
              service_data:
                media: station_google_home
                radio: NRJ
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state_not: NRJ
          type: conditional
        - card:
            image: /local/img/radio/Fun_selected.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.stop_radio
              service_data:
                media: station_google_home
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state: Fun
          type: conditional
        - card:
            image: /local/img/radio/Fun.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.start_radio
              service_data:
                media: station_google_home
                radio: Fun
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state_not: Fun
          type: conditional
        - card:
            image: /local/img/radio/Nostalgie_selected.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.stop_radio
              service_data:
                media: station_google_home
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state: Nostalgie
          type: conditional
        - card:
            image: /local/img/radio/Nostalgie.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.start_radio
              service_data:
                media: station_google_home
                radio: Nostalgie
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state_not: Nostalgie
          type: conditional
      type: horizontal-stack
  - cards: null
    type: conditional
    conditions:
      - entity: media_player.station_google_home
        state_not: 'off'
      - entity: media_player.station_google_home
        state_not: unavailable
    card:
      cards:
        - card:
            image: /local/img/radio/FG_selected.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.stop_radio
              service_data:
                media: station_google_home
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state: FG
          type: conditional
        - card:
            image: /local/img/radio/FG.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.start_radio
              service_data:
                media: station_google_home
                radio: FG
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state_not: FG
          type: conditional
        - card:
            image: /local/img/radio/Hitsradio_selected.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.stop_radio
              service_data:
                media: station_google_home
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state: Hitsradio
          type: conditional
        - card:
            image: /local/img/radio/Hitsradio.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.start_radio
              service_data:
                media: station_google_home
                radio: Hitsradio
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state_not: Hitsradio
          type: conditional
        - card:
            image: /local/img/radio/Scoop_selected.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.stop_radio
              service_data:
                media: station_google_home
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state: Scoop
          type: conditional
        - card:
            image: /local/img/radio/Scoop.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.start_radio
              service_data:
                media: station_google_home
                radio: Scoop
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state_not: Scoop
          type: conditional
        - card:
            image: /local/img/radio/MTI_selected.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.stop_radio
              service_data:
                media: station_google_home
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state: MTI
          type: conditional
        - card:
            image: /local/img/radio/MTI.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.start_radio
              service_data:
                media: station_google_home
                radio: MTI
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state_not: MTI
          type: conditional
        - card:
            image: /local/img/radio/BFM_selected.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.stop_radio
              service_data:
                media: station_google_home
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state: BFM
          type: conditional
        - card:
            image: /local/img/radio/BFM.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.start_radio
              service_data:
                media: station_google_home
                radio: BFM
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state_not: BFM
          type: conditional
        - card:
            image: /local/img/radio/Skyrock_selected.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.stop_radio
              service_data:
                media: station_google_home
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state: Skyrock
          type: conditional
        - card:
            image: /local/img/radio/Skyrock.jpg?v=1.0
            tap_action:
              action: call-service
              service: script.start_radio
              service_data:
                media: station_google_home
                radio: Skyrock
            type: picture
          conditions:
            - entity: input_select.radio_station_google_home
              state_not: Skyrock
          type: conditional
      type: horizontal-stack
```
/!\ station_google_home = le nom de votre entité media player
