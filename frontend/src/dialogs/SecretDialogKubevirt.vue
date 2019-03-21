<!--
Copyright (c) 2019 by SAP SE or an SAP affiliate company. All rights reserved. This file is licensed under the Apache Software License, v. 2 except as noted otherwise in the LICENSE file

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
 -->

<template>
  <secret-dialog
    :value=value
    :data="secretData"
    :dataValid="valid"
    :secret="secret"
    cloudProviderKind="kubevirt"
    color="blue darken-2"
    infraIcon="kubevirt"
    backgroundSrc="/static/background_kubevirt.svg"
    createTitle="Add new Kubevirt Cloud Secret"
    replaceTitle="Replace Kubevirt Cloud Secret"
    @input="onInput">

    <template slot="data-slot">
      <v-layout row>
        <v-flex xs8>
          <v-text-field
            color="blue darken-2"
            ref="namespace"
            v-model="namespace"
            :label="namespaceLabel"
            :error-messages="getErrorMessages('namespace')"
            @input="$v.namespace.$touch()"
            @blur="$v.namespace.$touch()"
            counter="64"
            hint="e.g. my-namespace"
          ></v-text-field>
        </v-flex>
      </v-layout>

      <v-layout row>
        <v-flex xs8>
          <v-textarea
            ref="kubeconfig"
            color="blue darken-2"
            box
            v-model="kubeconfig"
            :label="kubeconfigLabel"
            :error-messages="getErrorMessages('kubeconfig')"
            @input="$v.kubeconfig.$touch()"
            @blur="$v.kubeconfig.$touch()"
            hint="Enter a kubeconfig in YAML format"
            persistent-hint
          ></v-textarea>
        </v-flex>
      </v-layout>
    </template>

  </secret-dialog>

</template>

<script>
import SecretDialog from '@/dialogs/SecretDialog'
import { required, maxLength } from 'vuelidate/lib/validators'
import { getValidationErrors, setDelayedInputFocus } from '@/utils'

const validationErrors = {
  namespace: {
    required: 'You can\'t leave this empty.',
    maxLength: 'It exceeds the maximum length of 64 characters.'
  },
  kubeconfig: {
    required: 'You can\'t leave this empty.'
  }
}

export default {
  components: {
    SecretDialog
  },
  props: {
    value: {
      type: Boolean,
      required: true
    },
    secret: {
      type: Object
    }
  },
  data () {
    return {
      namespace: undefined,
      kubeconfig: undefined,
      validationErrors
    }
  },
  validations () {
    // had to move the code to a computed property so that the getValidationErrors method can access it
    return this.validators
  },
  computed: {
    valid () {
      return !this.$v.$invalid
    },
    secretData () {
      return {
        namespace: this.namespace,
        kubeconfig: this.kubeconfig
      }
    },
    validators () {
      const validators = {
        namespace: {
          required,
          maxLength: maxLength(64)
        },
        kubeconfig: {
          required
        }
      }
      return validators
    },
    isCreateMode () {
      return !this.secret
    },
    namespaceLabel () {
      return this.isCreateMode ? 'Namespace' : 'New Namespace'
    },
    kubeconfigLabel () {
      return this.isCreateMode ? 'Kubeconfig' : 'New Kubeconfig'
    }
  },
  methods: {
    onInput (value) {
      this.$emit('input', value)
    },
    reset () {
      this.$v.$reset()

      this.namespace = ''
      this.kubeconfig = ''

      if (!this.isCreateMode) {
        setDelayedInputFocus(this, 'namespace')
      }
    },
    getErrorMessages (field) {
      return getValidationErrors(this, field)
    }
  },
  watch: {
    value: function (value) {
      if (value) {
        this.reset()
      }
    }
  }
  // TODO:
  // mounted () {
  //   const onDrop = (value) => {
  //     this.kubeconfig = value
  //   }
  //   handleTextFieldDrop(this.$refs.kubeconfig, /yaml/, onDrop)
  // }
}
</script>
