Template Climate
================

.. seo::
    :description: Instructions for setting up template climate controllers with ESPHome.
    :image: description.svg

The ``template`` climate platform allows you to create a climate controller which combines the state
of other components.

.. code-block:: yaml

    # Example configuration entry
    climate:
      - platform: template
        name: "LG Therma"
        target_temperature_id: climate_target_temperature
        current_temperature_id: climate_current_temperature
        mode_id: climate_mode
        fan_mode_id: climate_fan_mode
        action_id: climate_action
        visual:
          temperature_step: 0.5C

Configuration variables:
------------------------

- **target_temperature_id** (*Required*, :ref:`Number <config-number>`):
  A Number which stores the target temperature.
- **current_temperature_id** (*Required*, :ref:`Sensor <config-sensor>`):
  A sensor to read the current temperature from.
- **mode_id** (*Required*, :ref:`Select <config-select>`):
  A select that configures the desired operation of the device.
  The referenced selects options must be a subset of the permitted climate modes.
- **fan_mode_id** (*Optional*, :ref:`Select <config-select>`):
  A select that determines the available and current fan mode of the device.
  The referenced selects options must be a subset of the permitted climate fan modes.
- **swing_mode_id** (*Optional*, :ref:`Select <config-select>`):
  A select that determines the available and current swing mode of the device.
  The referenced selects options must be a subset of the permitted climate swing modes.
- **preset_id** (*Optional*, :ref:`Select <config-select>`):
  A select that determines the available and current preset of the device.
  The referenced selects options must be a subset of the permitted preset modes modes.
- **action_id** (*Optional*, :ref:`TextSensor <config-text_sensor>`):
  A text sensor that returns the current operation state of the device.
- All other options from :ref:`Climate <config-climate>`.

.. _climate-template-select:

Templated Selects
-----------------

It may very well be that the Select of the device you are configuring does not map well to the
states required by the Climate component. In this case, you can choose to use a template Select to
route specific states to the right components:

.. code-block:: yaml

    select:
        # This select routes the OFF state to a switch, and the other states to a select.
      - platform: template
        id: climate_mode
        lambda: !lambda |-
            return id(lg_therma_active).state
                 ? id(lg_therma_mode).state
                 : std::string("OFF");
        update_interval: 1s
        set_action:
          - if:
              condition:
                lambda: 'return x != "OFF";'
              then:
                - switch.turn_on: lg_therma_active
                - select.set:
                    id: lg_therma_mode
                    option: !lambda return x;
              else:
                - switch.turn_off: lg_therma_active
        options:
          - "OFF"
          - "COOL"
          - "HEAT"
          - "AUTO"

        # This select handles the states for when the device is active.
      - platform: modbus_controller
        id: lg_therma_mode
        address: 0
        value_type: U_WORD
        optionsmap:
          "COOL": 0
          "HEAT": 4
          "AUTO": 5

    switch:
        # This switch determines whether the device is active.
      - platform: modbus_controller
        id: lg_therma_active
        modbus_controller_id: lg
        register_type: coil
        address: 0


See Also
--------

- :ref:`sensor-filters`
- :ref:`automation`
- :apiref:`template/sensor/template_sensor.h`
- :ghedit:`Edit`

