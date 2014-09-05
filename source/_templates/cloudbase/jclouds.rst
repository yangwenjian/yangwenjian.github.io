



=====================================
OpenStack SDK -- JClouds 
=====================================
JClouds is an SDK of common clouds platform.

Introduction
=====================================
JClouds是一个开源框架，它可帮你在云计算中起步并重用你的Java和Clojure开发技能。
JClouds的API允许你自由的使用可迁移的抽象或特定的云特性。我们支持多种云环境：Amazon, VMWare, Azure 和 Rackspace

Excercise
=====================================
We use jclouds as our SDK to build our own cloud system.

JClouds中的请求
-------------------------------------
JClouds作为Openstack的sdk，不外乎的作用就是将用户请求的接口转换为http的请求，然后将返回的json结果转换为结构化的对象。

JClouds中的token缓存机制
-------------------------------------
JClouds还有一个重要作用，就是将请求的token进行状态缓存。这是为了充分利用token的有效期，并能有效的减少请求次数。

将token请求后进行有效的存储。

::

    V value = null;
    try {
        value = getUninterruptibly(newValue);
        if (value == null) {
            throw new InvalidCacheLoadException("CacheLoader returned null for key " + key + ".");
        }
        statsCounter.recordLoadSuccess(loadingValueReference.elapsedNanos());
        storeLoadedValue(key, hash, loadingValueReference, value);
        return value;
    } finally {
        if (value == null) {
            statsCounter.recordLoadException(loadingValueReference.elapsedNanos());
            removeLoadingValue(key, hash, loadingValueReference);
        }
    }

同时还写了自动刷新token的代码，当请求token的时候，就请求scheduleRefresh方法，这个方法会返回一个有效的token。
这里使用一个时间记录标记now获得当前时间，然后将now值传入scheduleRfresh中，得到一个现在有效的token。

::

    if (count != 0) {
    // read-volatile
    // don't call getLiveEntry, which would ignore loading values
        ReferenceEntry<K, V> e = getEntry(key, hash);
        if (e != null) {
            long now = map.ticker.read();
            V value = getLiveValue(e, now);
            if (value != null) {
                recordRead(e, now);
                statsCounter.recordHits(1);
                return scheduleRefresh(e, key, hash, value, now, loader);
            }
            ValueReference<K, V> valueReference = e.getValueReference();
            if (valueReference.isLoading()) {
                return waitForLoadingValue(e, key, valueReference);
            }
        }
    }


JClouds将值存储之后进行定时刷新，如果时间超过定时刷新时间，就重新获取value并且返回，如果不是，那么就会返回oldValue。

::

    V scheduleRefresh(ReferenceEntry<K, V> entry, K key, int hash, V oldValue, long now,
        CacheLoader<? super K, V> loader) {
        if (map.refreshes() && (now - entry.getWriteTime() > map.refreshNanos)
            && !entry.getValueReference().isLoading()) {
            V newValue = refresh(key, hash, loader, true);
            if (newValue != null) {
                return newValue;
            }
        }
        return oldValue;
    }

