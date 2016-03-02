---
categories: GIS, GeoTools, devs, Java
date: 2006/06/30 18:46:38
guid: http://localhost/paolocorti/?p=5
permalink: http://www.paolocorti.net/2006/06/30/geometry-creation-with-geotools-2-java-merge-shapefile-polygons-on-a-common-attribute-polygon-geoprocessing/
tags: GIS, GeoTools, devs, Java
title: 'Geometry creation with GeoTools 2 (Java): merge shapefile polygons on a common
  attribute (polygon geoprocessing)'
---
Here I post a sample java class I developed some months ago, working with geometry creation in GeoTools 2. This class, given an input shapefile, will elaborate an output shapefile merging the input polygons on a common field.

<blockquote>Geotools is an open source (LGPL) Java code library which provides standards compliant methods for the manipulation of geospatial data, for example to implement Geographic Information Systems (GIS). The Geotools library implements Open Geospatial Consortium (OGC) specifications as they are developed, in close collaboration with the GeoAPI and GeoWidgets projects. The capabilities of Geotools are presented in the feature list.</blockquote>

$$code(lang=java)
/*
 * Created on 24-giu-2004
 */
package com.PaoloCorti.GIS.GeometryBuilder;
 
/**
 * @author corti
 * Richiede GeoTools2 e jts 1.4.0
 * Effettua il merge dei poligoni di uno shapefile in input sulla base di un campo, restituendo uno shapefile in output
 */
import java.io.FileInputStream;
import java.io.IOException;
import java.net.URL;
import java.nio.channels.FileChannel;
import java.util.Arrays;
import org.geotools.data.DataUtilities;
import org.geotools.data.FeatureReader;
import org.geotools.data.FeatureResults;
import org.geotools.data.FeatureSource;
import org.geotools.data.FeatureStore;
import org.geotools.data.Transaction;
import org.geotools.data.shapefile.ShapefileDataStore;
import org.geotools.data.shapefile.dbf.DbaseFileReader;
import org.geotools.feature.AttributeType;
import org.geotools.feature.AttributeTypeFactory;
import org.geotools.feature.Feature;
import org.geotools.feature.FeatureType;
import org.geotools.feature.FeatureTypeFactory;
import org.geotools.feature.IllegalAttributeException;
import org.geotools.filter.AttributeExpression;
import org.geotools.filter.CompareFilter;
import org.geotools.filter.FilterFactory;
import org.geotools.filter.FilterType;
import org.geotools.filter.LiteralExpression;
import com.vividsolutions.jts.geom.Geometry;
import com.vividsolutions.jts.geom.MultiPolygon;
 
public class UnionPolygonByAttribute {
 
 /**
  * main, richiede come parametri:
  * 1. shapefile di input (es: C:\temp\datiGIS\province.shp)
  * 2. campo sul quale effettuare il merge (es: c:\temp\datiGIS\output.shp)
  * 3. shapefile di output (es: COD_REG)
  * Es per lanciarlo da linea di comando:
  * java -classpath C:/java/geotools/jars/gt2-main.jar;C:/java/geotools/jars/gt2-shapefile.jar;C:/java/geotools/lib/JTS-1.4.jar; com.PaoloCorti.GIS.GeometryBuilder.UnionPolygonByAttribute "file:/C:/temp/datiGIS/province" "COD_REG" "file:/C:/temp/datiGIS/output"
  * @param args
  * @throws Exception
  */
      public static void main(String[] args) throws Exception  {
             if(args.length>0){
                  String inputShape = args[0];
                  String fieldToUnionName = args[1];
                  String outputShape = args[2];
                  String inDBFName = inputShape.substring(6, inputShape.length()) + ".dbf";
                  Stampa("processo lo shape " + inputShape + " nello shape " + outputShape + " unendo le feature con lo stesso " + fieldToUnionName);
                  UnionPolyByField(inputShape, outputShape, fieldToUnionName, inDBFName);
             }
             else
               Stampa("specificare tutti i parametri!");
      }
      
      /**
       * Effettua il merge dello shape di input in uno shape di output
       * @param InputShape Shapefile di input (da mergiare)
       * @param OutputShape Shapefile di output (mergiato)
       * @param FieldToUnionName Campo sul quale effettuare il merge nello shapefile di input
       * @param InDBFName Nome tabella DBF
       * @throws Exception
       */
      private static void UnionPolyByField(String InputShape, String OutputShape, String FieldToUnionName, String InDBFName) throws Exception
      {
            FileChannel in = new FileInputStream(InDBFName).getChannel();
                         DbaseFileReader r = new DbaseFileReader(in);
                         int fields = r.getHeader().getNumFields();
                         int recordCount = r.getHeader().getNumRecords();
                         int fieldToUnionIndex=0;
                         Class fieldType = null;
                         for (int i=0; i<fields; i++){
                             String fieldName = r.getHeader().getFieldName(i);
                             //Stampa(fieldName);
                             //Stampa(FieldToUnionName);
                             if(fieldName.equalsIgnoreCase(FieldToUnionName))
                             {
                                   fieldToUnionIndex = i;
                                   //leggiamo il datatype
                                   fieldType = r.getHeader().getFieldClass(i);
                                   break;
                             }
                         }
                         Stampa("tipo campo: " + fieldType.toString());
                         String fieldUniqueValueArr[] = new String[recordCount];
                         int j = 0;
                         while (r.hasNext()) {
                             DbaseFileReader.Row row = r.readRow();
                             //Stampa(row.read(fieldToUnionIndex).toString());
                             //make array
                             fieldUniqueValueArr[j]=row.read(fieldToUnionIndex).toString();
                             j++;
                         }
                         //shape da leggere e interrogare (input)
                         ShapefileDataStore sds = new ShapefileDataStore(new URL(InputShape));
                         String name = sds.getTypeNames()[0];  //solo 1 parametro per gli shape
                         Stampa("DATASTORE.getTypeNames() = " + name);
                         //leggo le feature
                         FeatureSource fs = sds.getFeatureSource(name);
                         FeatureType featureType = fs.getSchema();
                         FeatureResults fres = fs.getFeatures();
                         Stampa(fres.getCount() + " features nello shapefile");
                         //creiamo lo shapefile
                         ShapefileDataStore newShapeFileDatastore = new ShapefileDataStore(new URL(OutputShape));
                         //creiamo lo schema dello shape
                         AttributeType attUnionGeom = AttributeTypeFactory.newAttributeType("the_geom", MultiPolygon.class);
                         AttributeType attArea = AttributeTypeFactory.newAttributeType("AREA", Double.class);
                         AttributeType attPerimeter = AttributeTypeFactory.newAttributeType("PERIMETER", Double.class);
                         FeatureType newFeatureType = FeatureTypeFactory.newFeatureType(new AttributeType[] {attUnionGeom, attArea, attPerimeter}, name);
                         newShapeFileDatastore.createSchema(newFeatureType);
                         FeatureSource newFeatureSource = newShapeFileDatastore.getFeatureSource(name);
                         FeatureStore newFeatureStore = (FeatureStore)newFeatureSource;
                         //sort array
                         Arrays.sort(fieldUniqueValueArr);
                         String oldValue = fieldUniqueValueArr[0];
                         Stampa(oldValue);
                         for(int i=0; i<fieldUniqueValueArr.length; i++){
                           if(!fieldUniqueValueArr.equalsIgnoreCase(oldValue) || i==0){
                                oldValue = fieldUniqueValueArr;
                                Stampa(oldValue);
                                //faccio il filtro
                                    FilterFactory ff = FilterFactory.createFilterFactory();
                                    CompareFilter cf = ff.createCompareFilter(FilterType.COMPARE_EQUALS);
                                    AttributeExpression attExpression = ff.createAttributeExpression(featureType, FieldToUnionName);
                                    LiteralExpression l1;
                                    l1 = ff.createLiteralExpression(Double.parseDouble(oldValue)); //qui va modificato a seconda del datatype
                                    cf.addLeftValue(attExpression);
                                    cf.addRightValue(l1);
                                    Stampa(cf.toString());
                                    //faccio la query
                                    FeatureResults fQueryResults = fs.getFeatures(cf);
                                    int numFeat = fQueryResults.getCount();
                                    Stampa(numFeat + " features con " + FieldToUnionName + " = " + oldValue);
                                    //leggiamo le feature ottenute
                                    FeatureReader featureReader = fQueryResults.reader();
                                    //edit
                                    if(numFeat>0){
                                    Transaction t = newFeatureStore.getTransaction();
                                    //aggiungiamo le feature
                                    Geometry polyUnion = PolygonUnion(featureReader, numFeat);
                                    if(polyUnion instanceof MultiPolygon)
                                    {
                                          Double area = new Double(polyUnion.getArea());
                                          Double perimeter = new Double(polyUnion.getLength());
                                          Feature f = newFeatureType.create(new Object[] {polyUnion, area, perimeter});
                                          FeatureReader newFeatureReader = DataUtilities.reader(new Feature[] {f});
                                          newFeatureStore.addFeatures(newFeatureReader);
                                     }
                                    t.commit();
                                    t.close();
                                    }
                           }
                         }                      
      }
      
      /**
       * Unisce n poligoni in un poligono
       * @param fr
       * @param nfeat
       * @return
       */
      private static Geometry PolygonUnion(FeatureReader fr, int nfeat) {
            try {
                             //array Geometry jts
                             int i=0;
                             Geometry geomarr[] = new Geometry[nfeat];
                          while(fr.hasNext()) {
                               Feature feature = fr.next();
                               //Stampa(String.valueOf(feature.getDefaultGeometry().getArea())); 
                               geomarr[i++] = feature.getDefaultGeometry();
                             //System.out.print(i + "/" + nfeat + "-");
                          }
                             //ora faccio buffer
                             Geometry geom = geomarr[0];
                             for (i=0; i<nfeat; i++) {
                                   geom = geom.union(geomarr);
                             }
                             //Stampa(String.valueOf(String.valueOf(geom.getArea())));
                             Stampa("CREATO POLIGONO!");
                             return geom;
            }
            catch(IllegalAttributeException error){
                  Stampa("errore geometrico!");
                  return null;            
            }
            catch(IOException error){
                  Stampa("errore geometrico!");
                  return null;            
            }
      }
      
      private static void Stampa(String stringa){
                  System.out.println(stringa);
            }
}
$$/code


