# libsvm
This library uses emscripten to port [libsvm](https://github.com/cjlin1/libsvm) to asm and WebAssembly, for usage in the browser or nodejs.

libsvm version: 3.22


# What are asm and Webassembly ?
From [asmjs.org](http;//asmjs.org)
> asm is an optimizable subset of javascript.

From [webassembly.org](http://webassembly.org)
> WebAssembly or wasm is a new portable, size- and load-time-efficient format suitable for compilation to the web
# Should I use asm or WebAssembly ?
Both. You should try to use WebAssembly first and fall back to asm in order to support all browsers.

WebAssembly is currently supported in the latest stable versions of Chrome, Firefox.

# Benchmarks
Speed is mainly affected by the javascript engine that compiles it. Since WebAssembly has been stabilized and is an optimization phase, more recent engines are almost always faster.

Speed is also affected by the version of emscripten that generated the build or the options used in the build. I will try to keep up with any improvement that might significantly impact the performance.

<a name="SVM"></a>

## SVM
**Kind**: global class  

* [SVM](#SVM)
    * [new SVM(options)](#new_SVM_new)
    * _instance_
        * [.train(samples, labels)](#SVM+train)
        * [.crossValidation(samples, labels, kFold)](#SVM+crossValidation) ⇒ <code>Array.&lt;number&gt;</code>
        * [.free()](#SVM+free)
        * [.predictOne(sample)](#SVM+predictOne) ⇒ <code>number</code>
        * [.predict(samples)](#SVM+predict) ⇒ <code>Array.&lt;number&gt;</code>
        * [.getLabels()](#SVM+getLabels) ⇒ <code>Array.&lt;number&gt;</code>
        * [.getSVIndices()](#SVM+getSVIndices) ⇒ <code>Array.&lt;number&gt;</code>
        * [.serializeModel()](#SVM+serializeModel) ⇒ <code>string</code>
    * _static_
        * [.SVM_TYPES](#SVM.SVM_TYPES) : <code>Object</code>
        * [.KERNEL_TYPES](#SVM.KERNEL_TYPES) : <code>Object</code>
        * [.load(serializedModel)](#SVM.load) ⇒ [<code>SVM</code>](#SVM)

<a name="new_SVM_new"></a>

### new SVM(options)

| Param | Type | Default | Description |
| --- | --- | --- | --- |
| options | <code>object</code> |  |  |
| [options.type] | <code>number</code> | <code>SVM_TYPES.C_SVC</code> | Type of SVM to perform, |
| [options.kernel] | <code>number</code> | <code>KERNEL_TYPES.RBF</code> | Kernel function, |
| [options.degree] | <code>number</code> | <code>3</code> | Degree of polynomial, for polynomial kernel |
| [options.gamma] | <code>number</code> |  | Gamma parameter of the RBF, Polynomial and Sigmoid kernels. Default value is 1/num_features |
| [options.coef0] | <code>number</code> | <code>0</code> | coef0 parameter for Polynomial and Sigmoid kernels |
| [options.cost] | <code>number</code> | <code>1</code> | Cost parameter, for C SVC, Epsilon SVR and NU SVR |
| [options.nu] | <code>number</code> | <code>0.5</code> | For NU SVC and NU SVR |
| [options.epsilon] | <code>number</code> | <code>0.1</code> | For epsilon SVR |
| [options.cacheSize] | <code>number</code> | <code>100</code> | Cache size in MB |
| [options.tolerance] | <code>number</code> | <code>0.001</code> | Tolerance |
| [options.shrinking] | <code>boolean</code> | <code>true</code> | Use shrinking euristics (faster), |
| [options.probabilityEstimates] | <code>boolean</code> | <code>false</code> | weather to train SVC/SVR model for probability estimates, |
| [options.weight] | <code>object</code> |  | Set weight for each possible class |
| [options.quiet] | <code>boolean</code> | <code>true</code> | Print info during training if false |

<a name="SVM+train"></a>

### svM.train(samples, labels)
Trains the SVM model.

**Kind**: instance method of [<code>SVM</code>](#SVM)  
**Throws**:

- if SVM instance was instantiated from SVM.load.


| Param | Type | Description |
| --- | --- | --- |
| samples | <code>Array.&lt;Array.&lt;number&gt;&gt;</code> | The training samples. First level of array are the samples, second level are the individual features |
| labels | <code>Array.&lt;number&gt;</code> | The training labels. It should have the same size as the samples. If you are training a classification model, the labels should be distinct integers for each class. If you are training a regression model, each label should be the value of the predicted variable. |

<a name="SVM+crossValidation"></a>

### svM.crossValidation(samples, labels, kFold) ⇒ <code>Array.&lt;number&gt;</code>
Performs k-fold cross-validation (KF-CV). KF-CV separates the data-set into kFold random equally sized partitions,
and uses each as a validation set, with all other partitions used in the training set. Observations left over
from if kFold does not divide the number of observations are left out of the cross-validation process. If
kFold is one, this is equivalent to a leave-on-out cross-validation

**Kind**: instance method of [<code>SVM</code>](#SVM)  
**Returns**: <code>Array.&lt;number&gt;</code> - The array of predicted labels produced by the cross validation. Has a size equal to
the number of samples provided as input.  
**Throws**:

- if SVM instance was instantiated from SVM.load.


| Param | Type | Description |
| --- | --- | --- |
| samples | <code>Array.&lt;Array.&lt;number&gt;&gt;</code> | The training samples. |
| labels | <code>Array.&lt;number&gt;</code> | The training labels. |
| kFold | <code>number</code> | Number of datasets into which to split the training set. |

<a name="SVM+free"></a>

### svM.free()
Free the memory allocated for the model. Since this memory is stored in the memory model of emscripten, it is
allocated within an ArrayBuffer and WILL NOT BE GARBARGE COLLECTED, you have to explicitly free it. So
not calling this will result in memory leaks. As of today in the browser, there is no way to hook the
garabage collection of the SVM object to free it automatically.
Free the memory that was created by the compiled libsvm library to.
store the model. This model is reused every time the predict method is called.

**Kind**: instance method of [<code>SVM</code>](#SVM)  
<a name="SVM+predictOne"></a>

### svM.predictOne(sample) ⇒ <code>number</code>
Predict the label of one sample.

**Kind**: instance method of [<code>SVM</code>](#SVM)  
**Returns**: <code>number</code> - - The predicted label.  

| Param | Type | Description |
| --- | --- | --- |
| sample | <code>Array.&lt;number&gt;</code> | The sample to predict. |

<a name="SVM+predict"></a>

### svM.predict(samples) ⇒ <code>Array.&lt;number&gt;</code>
Predict the label of many samples.

**Kind**: instance method of [<code>SVM</code>](#SVM)  
**Returns**: <code>Array.&lt;number&gt;</code> - - The predicted labels.  

| Param | Type | Description |
| --- | --- | --- |
| samples | <code>Array.&lt;Array.&lt;number&gt;&gt;</code> | The samples to predict. |

<a name="SVM+getLabels"></a>

### svM.getLabels() ⇒ <code>Array.&lt;number&gt;</code>
Get the array of labels from the model. Useful when creating an SVM instance with SVM.load

**Kind**: instance method of [<code>SVM</code>](#SVM)  
**Returns**: <code>Array.&lt;number&gt;</code> - - The list of labels.  
<a name="SVM+getSVIndices"></a>

### svM.getSVIndices() ⇒ <code>Array.&lt;number&gt;</code>
Get the indices of the support vectors from the training set passed to the train method.

**Kind**: instance method of [<code>SVM</code>](#SVM)  
**Returns**: <code>Array.&lt;number&gt;</code> - - The list of indices from the training samples.  
<a name="SVM+serializeModel"></a>

### svM.serializeModel() ⇒ <code>string</code>
Uses libsvm's serialization method of the model.

**Kind**: instance method of [<code>SVM</code>](#SVM)  
**Returns**: <code>string</code> - The serialization string.  
<a name="SVM.SVM_TYPES"></a>

### SVM.SVM_TYPES : <code>Object</code>
SVM classification and regression types

**Kind**: static property of [<code>SVM</code>](#SVM)  
**Properties**

| Name | Description |
| --- | --- |
| C_SVC | The C support vector classifier type |
| NU_SVC | The nu support vector classifier type |
| ONE_CLASS | The one-class support vector classifier type |
| EPSILON_SVR | The epsilon support vector regression type |
| NU_SVR | The nu support vector regression type |

<a name="SVM.KERNEL_TYPES"></a>

### SVM.KERNEL_TYPES : <code>Object</code>
SVM kernel types

**Kind**: static property of [<code>SVM</code>](#SVM)  
**Properties**

| Name | Description |
| --- | --- |
| LINEAR | Linear kernel |
| POLYNOMIAL | Polynomial kernel |
| RBF | Radial basis function (gaussian) kernel |
| SIGMOID | Sigmoid kernel |

<a name="SVM.load"></a>

### SVM.load(serializedModel) ⇒ [<code>SVM</code>](#SVM)
Create a SVM instance from the serialized model.

**Kind**: static method of [<code>SVM</code>](#SVM)  
**Returns**: [<code>SVM</code>](#SVM) - - SVM instance that contains the model.  

| Param | Type | Description |
| --- | --- | --- |
| serializedModel | <code>string</code> | The serialized model. |
