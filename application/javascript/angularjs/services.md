# Application Architecture | AngularJS | Services 

## Pattern

```js
(function (angular) {
    'use strict';

    /* Define the service events */
    var svcEvents = { valueUpdated: 'AngularService-valueUpdated' };

    /* Define the service logic */
    var svcDefinition = ['$rootScope', '$timeout'];
    function AngularService($rootScope, $timeout) {
        /* internal fields */
        var _sampleInternalVariable = 'Hello World!';

        /* internal methods */
        function _initializeService() { }

        /* exposed methods */
        function getSampleVariable() { return _sampleInternalVariable; }
        function setSampleVariable(param) {
            // validate
            param = typeof param === 'object' ? param : {};
            param.value = typeof param.value === 'string' ? param.value : '';
            // process
            _sampleInternalVariable = param.value;
            // broadcast
            $rootScope.$broadcast(svcEvents.valueUpdated, param);
        }

        /* finalize service */
        $timeout(_initializeService, 500);
        return {
            getSampleVariable: getSampleVariable,
            setSampleVariable: setSampleVariable
        };
    }

    /* Finalize the service setup */
    svcDefinition.push(AngularService);
    angular.module('AngularModule')
        .constant('AngularServiceEvents', svcEvents)
        .service('AngularService', svcDefinition);
})(window.angular);
```
