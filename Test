package com.sbi.epay.hazelcast.service;


import com.hazelcast.core.HazelcastInstance;
import com.hazelcast.core.HazelcastJsonValue;
import com.hazelcast.map.IMap;
import com.hazelcast.map.QueryResultSizeExceededException;
import com.hazelcast.map.impl.query.QueryResultCollection;
import com.hazelcast.query.Predicates;
import com.sbi.epay.hazelcast.constants.HazelcastConstants;

import com.sbi.epay.hazelcast.exception.HazelcastException;
import com.sbi.epay.hazelcast.model.CacheableEntity;
import com.sbi.epay.hazelcast.model.EPayCachebleData;
import com.sbi.epay.hazelcast.model.HazelCastJsonValueData;
import com.sbi.epay.hazelcast.model.PredicateInput;

import com.sbi.epay.logging.utility.LoggerFactoryUtility;
import com.sbi.epay.logging.utility.LoggerUtility;
import org.springframework.stereotype.Service;

/**
 * This class is created for cache operations
 * <p>
 * Class Name: HazelcastService
 * *
 * Description:This class is used for cache operations
 * *
 * Author: V1018212(Hrishikesh Pandirakar)
 * Copyright (c) 2024 [State Bank of India]
 * All rights reserved
 * *
 * Version:1.0
 */
@Service
public class HazelcastService {
    /**
     * This method will be used for adding data to cache
     *
     * @param inputObject it is an instance of CacheableEntity as input.
     * @param hazelcastInstance an instance of Hazelcast
     * @return a success String if data added to cache.
     */
    private static final LoggerUtility log= LoggerFactoryUtility.getLogger(HazelcastService.class);
    public  String addDataToCache(CacheableEntity inputObject,HazelcastInstance hazelcastInstance) throws HazelcastException {
        try {
            log.debug("HazelcastService :: addDataToCache ");
            IMap<String, Object> myMap = hazelcastInstance.getMap(inputObject.getMapName());
            myMap.put(inputObject.getKey(), inputObject.getCacheableEntityData());
            return HazelcastConstants.DATA_ADDED;
        } catch (NullPointerException e) {
            log.error("HazelcastService :: addDataToCache "+e.getMessage());
            throw new HazelcastException(HazelcastConstants.HAZELCAST_1001, HazelcastConstants.HAZELCAST_1001_MSG);
        }
    }

    /**
     * This method will be used for fetching data from cache using key.
     *
     * @param mapName it is a String which representing  name of a map where we have stored the data.
     * @param key     it a String against which we will fetch a specific data related to key.
     * @param hazelcastInstance an instance of Hazelcast
     * @return ResponseDto .
     */
    public  CacheableEntity getDataByKey(String mapName, String key,HazelcastInstance hazelcastInstance) throws HazelcastException {
        try {
            log.debug("HazelcastService :: getDataByKey ");
            IMap<String, Object> myMap = hazelcastInstance.getMap(mapName);
            return CacheableEntity.builder().key(key).mapName(mapName).cacheableEntityData((EPayCachebleData) myMap.get(key)).build();
        } catch (NullPointerException e) {
            log.error("HazelcastService :: getDataByKey "+e.getMessage());
            throw new HazelcastException(HazelcastConstants.HAZELCAST_1002, HazelcastConstants.HAZELCAST_1002_MSG);
        }

    }

    /**
     * This method will be used for fetching data from cache using SQL condition.
     *
     * @param mapName      it is a String which representing a name of a map where we have stored the data.
     * @param sqlCondition it a String which we will use as a SQL condition to fetch a data.
     * @param hazelcastInstance an instance of Hazelcast
     * @return ResponseDto .
     */
    public  CacheableEntity getDataBySql(String mapName, String sqlCondition,HazelcastInstance hazelcastInstance) throws HazelcastException {
        try {
            log.debug("HazelcastService :: getDataBySql ");
            IMap<String, Object> myMap = hazelcastInstance.getMap(mapName);
            return CacheableEntity.builder().mapName(mapName).responseData(myMap.values(Predicates.sql(sqlCondition))).build();
        } catch (NullPointerException e) {
            log.error("HazelcastService :: getDataBySql "+e.getMessage());
            throw new HazelcastException(HazelcastConstants.HAZELCAST_1003, HazelcastConstants.HAZELCAST_1003_MSG);
        }
    }

    /**
     * This method will be used for fetching data from cache using Predicate.
     *
     * @param predicateInput it is an Object of PredicateInput which contains a name of a map where we have stored the data and a Predicate instance.
     * @param hazelcastInstance an instance of Hazelcast
     * @return ResponseDto .
     */
    public  CacheableEntity getDataByPredicate(PredicateInput predicateInput,HazelcastInstance hazelcastInstance) throws HazelcastException {
        try {
            log.debug("HazelcastService :: getDataByPredicate ");
            IMap<String, Object> myMap = hazelcastInstance.getMap(predicateInput.getMapName());
            QueryResultCollection queryResultCollection = (QueryResultCollection) myMap.values(predicateInput.getPredicate());
            EPayCachebleData ePayCacheableData = (EPayCachebleData) queryResultCollection.stream().iterator().next();
            return CacheableEntity.builder().key(predicateInput.getKey()).mapName(predicateInput.getMapName()).cacheableEntityData(ePayCacheableData).build();
        } catch (QueryResultSizeExceededException |NullPointerException e) {
            log.error("HazelcastService :: getDataByPredicate "+e.getMessage());
            throw new HazelcastException(HazelcastConstants.HAZELCAST_1004, HazelcastConstants.HAZELCAST_1004_MSG);
        }
    }

    /**
     * This method will be used for adding data into cache directly in JSON format.
     *
     * @param mapName    it is a String which representing a name of a map where we have stored the data.
     * @param key        it a String against which we will fetch a specific data related to key
     * @param jsonString it a JSON String which we will store into cache.
     * @param hazelcastInstance an instance of Hazelcast
     * @return ResponseDto.
     */
    public  String saveDataByJsonObject(String mapName, String key, String jsonString,HazelcastInstance hazelcastInstance) throws HazelcastException {
        try {
            log.debug("HazelcastService :: saveDataByJsonObject ");
            IMap<String, HazelcastJsonValue> myMap = hazelcastInstance.getMap(mapName);
            myMap.put(key, new HazelcastJsonValue(jsonString));
            return HazelcastConstants.DATA_ADDED;
        } catch (NullPointerException e) {
            log.error("HazelcastService :: saveDataByJsonObject "+e.getMessage());
            throw new HazelcastException(HazelcastConstants.HAZELCAST_1005, HazelcastConstants.HAZELCAST_1005_MSG);
        }
    }

    /**
     * This method will be used for adding data into cache directly in JSON format.
     *
     * @param mapName    it is a String which representing a name of a map where we have stored the data.
     * @param key        it a String against which we will fetch a specific data related to key
     * @param hazelcastInstance an instance of Hazelcast
     * @return ResponseDto.
     */
    public  CacheableEntity getJSONData(String mapName, String key, HazelcastInstance hazelcastInstance) throws HazelcastException {
        try {
            log.debug("HazelcastService :: getJSONData ");
            IMap<String, HazelcastJsonValue> myMap = hazelcastInstance.getMap(mapName);
            return CacheableEntity.builder().key(key).mapName(mapName).hazelCastJsonValueData(new HazelCastJsonValueData(myMap.get(key))).build();
        } catch (Exception e) {
            log.error("HazelcastService :: getJSONData "+e.getMessage());
            throw new HazelcastException(HazelcastConstants.HAZELCAST_1006, HazelcastConstants.HAZELCAST_1006_MSG);
        }
    }

    /**
     * This method will be used for removing data from cache.
     *
     * @param mapName it is a String which representing a name of a map where we have stored the data.
     * @param key     it a String against which we will fetch a specific data related to key
     * @param hazelcastInstance an instance of Hazelcast
     * @return success String.
     */
    public  String removeData(String mapName, String key,HazelcastInstance hazelcastInstance) throws HazelcastException {
        try {
            log.debug("HazelcastService :: removeData ");
            IMap<String, HazelcastJsonValue> myMap = hazelcastInstance.getMap(mapName);
            myMap.remove(key);
            return HazelcastConstants.DATA_REMOVED;
        } catch (Exception e) {
            log.error("HazelcastService :: removeData "+e.getMessage());
            throw new HazelcastException(HazelcastConstants.HAZELCAST_1007, HazelcastConstants.HAZELCAST_1007_MSG);
        }
    }

    /**
     * This method will be used for update data of a map with specific key present in cache.
     *
     * @param inputObject it is an object of CacheableEntity which contains a name of a map where we have stored the data and CacheableEntity object
     * @param hazelcastInstance an instance of Hazelcast
     * @return success String.
     */
    public  String updateData(CacheableEntity inputObject,HazelcastInstance hazelcastInstance) throws HazelcastException {
        try {
            log.debug("HazelcastService :: updateData ");
            IMap<String, Object> myMap = hazelcastInstance.getMap(inputObject.getMapName());
            myMap.put(inputObject.getKey(), inputObject.getCacheableEntityData());
            return HazelcastConstants.DATA_UPDATE;
        } catch (Exception e) {
            log.error("HazelcastService :: updateData "+e.getMessage());
            throw new HazelcastException(HazelcastConstants.HAZELCAST_1008, HazelcastConstants.HAZELCAST_1008_MSG);
        }
    }

    /**
     * This method will be used for update data of a map with specific key present in cache.
     *
     * @param mapName it is a String which representing a name of a map where we have stored the data.
     * @param key     it a String against which we will fetch a specific data related to key
     * @param hazelcastInstance an instance of Hazelcast
     * @return success String.
     */
    public  String updateJsonData(String mapName, String key, String jsonString, HazelcastInstance hazelcastInstance) throws HazelcastException {
        try {
            log.debug("HazelcastService :: updateJsonData ");
            IMap<String, Object> myMap = hazelcastInstance.getMap(mapName);
            myMap.put(key, new HazelcastJsonValue(jsonString));
            return HazelcastConstants.DATA_UPDATE;
        } catch (Exception e) {
            log.error("HazelcastService :: updateJsonData "+e.getMessage());
            throw new HazelcastException(HazelcastConstants.HAZELCAST_1009, HazelcastConstants.HAZELCAST_1009_MSG);
        }
    }
}
